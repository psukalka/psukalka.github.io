+++
date = '2025-05-16T22:38:25+05:30'
draft = false
title = 'The Swiss Army Knife'
+++
FFmpeg is called the Swiss Army Knife for video and audio. Whatever you want to do with media like compressing, scaling, editing, encoding, decoding, transcoding, streaming, ... FFmpeg can do everything. If it's so special, let's start using it. You can install [FFmpeg](https://ffmpeg.bufferhead.com/) on your system.

Let's download some video `video_1.mp4` . I want to get details about this video. I can do so using: 
```
ffprobe -v quiet -print_format json -show_format -show_streams video_1.mp4
```
Here we used `ffprobe` (that usually comes with ffmpeg). 
- We specified it to print output in `json` format
- We asked it to display `format` of media. Format is the container in which the video is packaged. Like MP4 is a format
- We asked it to display `streams`. The file can contain multiple streams like video stream, audio stream, subtitle stream, ... 

Using the above command we can get all details of file like its duration, size, packaging, stream, bitrate, sample rate, ... 

The size of video file that I have is 200MB and its duration is just around 2 min. Its bitrate is around 10Mbps. If I want to stream such file or want my user to download this file, it would consume a lot of their data. How can I use ffmpeg to reduce size of this video ? 

```
ffmpeg -i video_2.mp4 -c:v libx264 -profile:v main -level 3.1 -b:v 800k -maxrate 1M -bufsize 1.5M -c:a aac 128k output.mp4
```
What we have done above is: 
- take an `input` video `video_2.mp4` 
- `-c:v` code the video stream with libx264 i.e. H264 codec 
- Use `main` profile while encoding at 3.1 level
- With `-b:v` we want the bitrate of video stream to be `average` 800k, `max` 1 million and we can use a buffer of upto 1.5 million
- Similarly for audio stream `-c:a` we are specifying `aac` codec and 128k bitrate
- the final output will be stored in `output.mp4`

Cool. I have now a video that has at 1Mbps bitrate. Since original bitrate was at 10Mbps and size was around 200MB, the size of this output file will be around 20MB 

I can use this file for streaming. But there is one problem here. User won't be able to seek the video at unbuffered location. Meaning that if 10 sec video is buffered user can't go to 1 min location. User will have to wait till 1 min of video is buffered. So, how do we enable this seeking ?

For this, we need to break video in smaller chunks (say of 3 sec). There is a special HLS (HTTP Live Streaming) format for this. The smaller chunks are transport stream (.ts) files. And to collate all these chunks we have a `.m3u8` playlist file that has mapping of which chunk to play and when. Let's create hls stream of `video_2.mp4` with ffmpeg. 

```
ffmpeg -i video_2.mp4 -c:v libx264 -profile:v high -level 3.1 -b:v 800k -maxrate 1M -bufsize 1.5M -start_number 0 -hls_time 3 -hls_list_size 0 -f hls playlist.m3u8
```
So, we have told ffmpeg to
- start from 0th position `-start_number 0`
- make chunks of 3 sec `-hls_time 3`
- keep on making chunks till file completes `hls_list_size 0` 
- and place the output in `playlist.m3u8` file

If duration of file is 2 min and chunk size is 3 sec, it would roughly make 40 `.ts` files. 

There is a little problem with this. ffmpeg takes `3` sec as a rough boundary. There is not a hard cut at 3 seconds. If we want chunks to be of exactly 3 sec, we can enforce it as below: 

```
ffmpeg -i video_2.mp4 -c:v libx264 -profile:v high -level 3.1 -b:v 800k -maxrate 1M -bufsize 1.5M -force_key_frames expr:gte(t,n_forced*3) -start_number 0 -hls_time 3 -hls_list_size 0 -hls_flags split_by_time -f hls playlist.m3u8
```
- we have flagged hls to `split_by_time`
- and forced each frame to have a key frame

This is essential in compression. If your chunk does not have a key-frame then that chunk can't be played independently. So, the video player will have to load previous chunk leading to longer wait time and bad user experience. 

This is just the tip of the iceberg. You can do n-number of things with ffmpeg. You can create a clip generator. You can add watermark to your videos. You can transcode video to other formats like `dash`. You can extract frames out of video to create video preview. There is a lot to it. That's why it is called a Swiss Army Knife. 