---
layout: post
title: Building a "Lava Lamp" using a Raspberry Pi
subtitle: A decorative RGB Matrix Display
gh-repo: mnemocron/my-discrete-fpga
tags: [raspberrypi, rgb]
comments: true
author: Simon
---

Recent recordings of my favorite genre (DnB) at a genius location (Printworks) inspired something in me. 
And since the Printworks location was recently demolished I decided to bring its visual magic into my home.

RGB is fantastic if done right! Adafruit has been selling large RGB panels for a while now. 
A while ago I tested it with an MCU and it has been collecting dust for a while - no use anymore.
My vision for this project was clear: I wanted to play video loops on a large matrix display with an unusual aspect ratio.

From Aliexpress I ordered a batch of 4x 64x64 RGB matrix display panels along with a 5V 200W power supply.
Furthermore, a Raspberry Pi Zero WH and a HUB75 adapter from a shady Chinese website that never arrived - and then an Adafruit RGB Matrix + RTC hat.

### Build process



### Important commands


**How to download only 1min from a 10h youtube Video**

```
yt-dlp "https://www.youtube.com/watch?v=y9PTNTSpGJs" -f 'bestvideo[height<=360]' --external-downloader ffmpeg --external-downloader-args "-ss 00:00:00 -to 00:01:00" -o "%(title)s_method2.%(ext)s"
```

**How to crop and rotate a video to match aspect ratio of RGB matrix?**

(you may want to do outsource this to another computer because the Pi Zero performs encoding only at ~5 fps)

```
ffmpeg -i water-full.webm -vf "crop=90:360:300:0,transpose=2,scale=trunc(oh*a/2)*2:64" -pix_fmt gbrp water.webm
```

```
ffmpeg -i water-full.webm -vf "crop=90:360:150:0,transpose=2,scale=trunc(oh*a/2)*2:64" -pix_fmt gbrp water.webm
```

- ffmpeg List supported pixel formats
https://lists.ffmpeg.org/pipermail/ffmpeg-devel/2007-May/035617.html

- Crop a video with ffmpeg
https://video.stackexchange.com/questions/4563/how-can-i-crop-a-video-with-ffmpeg

- Rotate a video with ffmpeg
https://stackoverflow.com/questions/3937387/rotating-videos-with-ffmpeg


