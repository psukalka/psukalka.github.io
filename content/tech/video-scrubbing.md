+++
date = '2025-05-08T20:32:53+05:30'
draft = false
title = 'Video Scrubbing'
+++
While watching YouTube video, if you hover over the seek bar you can preview video image of that location. 
You can see images even at 10 min even though only 30 sec of video has buffered yet. 
So, if the video has not loaded till 10 min, how is the video player able to display image at that location ? Isn't it interesting ?

This happens with a feature called video scrubbing. We need to process the uploaded video for video scrubbing. 
When a video is uploaded (say of 10 min), we can take snapshots at regular intervals (of say 3sec). 
So, for a 10 min video there will be 10*60/3 = 200 snapshots. We can generate these snapshots using ffmpeg. 

```
ffmpeg -i <input_video_path> -vf fps=1/3,scale=160:90 -q:v 3 <output_dir>/thumb-%04d.jpg
```

This would generate images of 160:90 size from input video at 3 sec interval with quality of 3 in the output directory. 
Output thumbnails will follow thumb-0001.jpg like pattern.

Loading 200 images would result in 200 requests from client. This would be very inefficient. Plus, the size of images is quite small (few kBs). If we can combine these images in some way, then we can reduce the number of requests from 200 to 1. 

There is a way. We can create a `sprite-sheet` from these generated images. Sprite sheet is a sheet of small small images. To identify which image is stored at which location a metadata.json should also be created. With this metadata.json the video player can easily display appropriate image when user hovers over some specific duration of video. 

We can use Pillow library to generate sprite-sheet as below: 
```
def create_sprite_sheet(
        thumbnail_paths: List[Path],
        output_path: Path,
        columns: int,
        thumbnail_width: int,
        thumbnail_height: int,
        interval_seconds: int
) -> Tuple[Path, List[Dict]]:
    """
    Create a sprite sheet from individual thumbnails.
    
    Args:
        thumbnail_paths (List[Path]): List of paths to thumbnail images
        output_path (Path): Path to save the sprite sheet
        columns (int): Number of thumbnails per row
        thumbnail_width (int): Width of each thumbnail in pixels
        thumbnail_height (int): Height of each thumbnail in pixels
        interval_seconds (int): Interval between thumbnails in seconds
    
    Returns:
        Tuple[Path, List[Dict]]: Path to the sprite sheet and list of frame metadata
    """
    try:
        from PIL import Image
    except ImportError:
        raise ImportError("Pillow is required to create sprite sheets. Install with 'pip install Pillow'")

    print(f"Creating sprite sheet with {len(thumbnail_paths)} thumbnails")

    # Calculate dimensions
    rows = math.ceil(len(thumbnail_paths) / columns)
    sprite_width = columns * thumbnail_width
    sprite_height = rows * thumbnail_height

    # Create a new image for the sprite sheet
    sprite_sheet = Image.new('RGB', (sprite_width, sprite_height))

    # Prepare metadata for frames
    frames_metadata = []

    # Add each thumbnail to the sprite sheet
    for i, thumb_path in enumerate(thumbnail_paths):
        # Calculate position
        row = i // columns
        col = i % columns
        x = col * thumbnail_width
        y = row * thumbnail_height

        # Open thumbnail and paste into sprite sheet
        with Image.open(thumb_path) as thumb:
            sprite_sheet.paste(thumb, (x, y))

        # Add metadata for this frame
        frames_metadata.append({
            "timestamp": i * interval_seconds,
            "position": {
                "x": x,
                "y": y
            },
            "index": i
        })

    # Save sprite sheet
    sprite_sheet.save(output_path, quality=85, optimize=True)
    print(f"Sprite sheet created at {output_path}")

    return output_path, frames_metadata
```
Sample sprite sheet image:
![Sprite Sheet](/images/sprite_sheet.jpg)

The json file looks like: 
```
  "frames": [
    {
      "timestamp": 0,
      "position": {
        "x": 0,
        "y": 0
      },
      "index": 0
    },
    {
      "timestamp": 3,
      "position": {
        "x": 160,
        "y": 0
      },
      "index": 1
    },
    ...
  ]
```

Once sprite-sheet and json metadata is created video player can use them to display appropriate images on hovering over seek bar. This is how video scrubbing works.