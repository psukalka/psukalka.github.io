+++
date = '2025-05-18T00:56:39+05:30'
draft = false
title = 'Understanding Video'
+++
Say, I have a `video.mp4` file on my system. What exactly does it contain ? The outermost layer is called `Container`. There are various type of containers `mp4`, `mkv`, `mov`, `flv`, ... Container contains metadata about the file like its title, duration, creation date, ... Then it contains streams of data. They can be video streams, audio streams, subtitle streams, ... This is pictorially explained in following image: 

![Video Container](/images/video-container.svg)

---
As is obvious, the majority of the size comes from video stream. Audio, subtitles and metadata occupy relatively smaller size. So, if we want to compress a video, we need to understand the video stream. 

What does a video stream contain ? At the very basic it is collection of frames. You must have heard 30fps, 60fps video. 30fps just means there are 30 frames in 1 sec of video. So, if a video has duration of 1 min (i.e. 60 sec), it will have 30*60 = 1800 frames. 

Cool, let's dig deeper into these frames. A frame is similar to an image (for simplicity). So, in a 30fps video, there will be 30 images in one second. These frames having complete information are called `I-Frames` (Intra-coded Frames). In a second what would change in the scene ? Hardly few things will move. So, as you might have assumed two adjacent frames F1 and F2 will slightly differ. So, instead of storing the complete information through I-frames, if we store F1 as I-Frame and F2 contains only diff from F1, then we can save some data. This frame containing just diff from previous frame is called `P-Frame` (Predicted Frame). If we have the capability to look both backward and forward (bi-direction) then we can be even more efficient with diff. This is the third kind of frame `B-Frame` (Bi-directional) that is most compact and depends on both previous and next frame. 

For a 10Mbps video, the sizes of these frames are as roughly follows:
- I-frame : 100-300KB
- P-frame : 30-100KB
- B-frame : 10-50KB

We can check the number of each type of frames in a video with following command: 
```
ffprobe -v quiet -select_streams v:0 -show_entries frame=pict_type -of csv=p=0 video.mp4 | sort | uniq -c
```

Now, suppose I want to compress a video. What is the maximum compression that I can achieve ? With greedy algorithm, I can choose maximum of B-frames, then P-frame and least number of I-frames. For a 1 min video, I will have `I-frame` at the start and then follow it all with `B-Frames` and have a `P-frame` at last. Thus, I will achieve maximum compression. But it would come at a cost. With each passing frame the quality of video will degrade and after a certain frame it will not even be recognizable. When there is a scene change, two frames will have significant diff. If we don't have any I-frame nearby quality of scene change will decrease drastically. Also, to draw with P/B frame you need to depend on previous I-frame. Thereotically you can make millions of P frames but codecs have memory limitations. They can store upto max 200-500 frames in memory for decoding. So, you can't have all P/B frames. In practice, there can be 60-120 P/B frames between 2 I-frames. This is called GOP-size. 

*As you can guess, if we want more compressed video (for low network latency), we should increase GOP-size to like 120. And if we want high quality video we should choose low GOP-size like 60.*

Say, I want to set gop size of 120 for a video. We can use following command to set `gop` size of a video. 
```
ffmpeg -i video.mp4 -c:v libx264 -g 120 -c:a copy output_gop.mp4
```

---
Cool, what else can we do ? A video stream is collection of frames. A 30fps video has 30 frames in one second. What if I reduce framerate, i.e. number of frames in 1 sec. Say I change framerate from 30fps to 24fps what will happen ? *We have reduced size of video roughly by 20%*. But what impact does it have on video quality ? It depends on content in the video. If it is a standard video, framerate change will not be noticeable. For slow paced content like news reporting we can even lower the framerate. But for a fast motion / action / sports video the change might be noticeable. But if your end-client is already in low network area, reducing framerate can be an option (because playing video is important than not playing at all). 

To change framerate of a video, we can use following command: 
```
ffmpeg -i video.mp4 -c:v libx264 -r 24 -preset medium -crf 18 -c:a copy output_24fps.mp4
```
---

Next, what if we reduce bitrate of video ? Instead of 10Mbps we restrict it max to 1Mbps. Codec algorithms will internally try to optimize frame types (I/P/B). Pixel values were be approximated and stored using lesser number of bits. Ex instead of storing color between 0-255, it can store it using 0-63. For smaller device (like mobile phones) such change won't be that noticeable. 

We can update bitrate of a video as follows:
```
ffmpeg -i video.mp4 -c:v libx264 -b:v 1M -c:a copy output_1mbps.mp4
```

---
Then, what if we play with resolution of video ? Say, original video is of 1080x1920 resolution and we change its resolution to 540x960 ? Here, we have kept the aspect ratio same. But the number of pixels have reduced by to 1/4th i.e. 75% reduction. Size will be drastically lesser. But what will be impact on quality ? On most mobile devices, the change will be imperceptible but edges or details will start getting softer. 

To change resolution of a video, we can use following command:
```
ffmpeg -i video.mp4 -vf "scale=960:540" -c:v libx264 -crf 18 -preset medium -c:a copy output_res.mp4
```
Most things are self-explanatory in above command. `crf` means Constant Rate Factor. Its value 18 is called quantization factor. Lower quantization factor means higher quality as more details are preserved. Value till 20 is perceived lossless to visual perception.

---
Cool. But can we do anything to preserve quality of video while lowering its size ? Yes. A video can contain scenes of various complexity. Some scenes will be simple like people talking, slow moving scenes, nature scene, ... Some scenes can be complex like an action sequence. If we use same number of bits to represent both simple and complex scenes, we would waste bits on simple scenes and complex scene will be of lower quality. If we are catering to a video that will be compressed once and watched 1000s of times later we can think of using multi-pass compression. In first pass, it codec will just iterate over video and create a log file analyzing locations of simple and complex scenes. In second pass, it will use this log file to allocate lesser bits (say 600kb) for simple scene and more bits (say 1400kb) for complex scene. Thus, overall average bitrate hasn't changed but perceivable quality is better. Multi-pass algorithm takes more time to process as it iterates multiple time, but it will produce better quality outputs at lesser size. 

To run multi-pass encoding we can use following command:
```
ffmpeg -i video.mp4 -c:v libx264 -b:v 1M -pass 1 -an -f null /dev/null
ffmpeg -i video.mp4 -c:v libx264 -b:v 1M -pass 2 -c:a copy output_multi.mp4
```
---
Last but quite important, using newer codecs (like H265 instead of H264) can have better impacts. H264 uses macroblocks of 16x16 pixels while H265 uses coding tree units (CTU) of upto 64x64 pixels which is more efficient for larger areas with similar content. H264 has 9 intra-prediction modes while H265 has 35 intra-predition models thus giving precise directional prediction. H265 also utilizes parallel processing for better core-utilization. Newer codec will obviously have better impact, just need to check that the endpoints support decoding them. 

To change codec from H264 to H265 we can use following command:
```
ffmpeg -i video.mp4 -c:v libx265 -crf 25 -c:a aac -b:a 128k output_h265.mp4
```
---
Thus, by understanding a video and using simple logical reasoning we can achieve desirable compression levels.