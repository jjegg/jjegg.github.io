---
title: 'Timelapse with ffmpeg'
date: 2023-01-01 00:00:00
description: How to create timelapse with ffmpeg.
featured_image: '/images/2017-nyc/01.jpg'
---

![](/images/demo/demo-landscape.jpg)

This tutorial is based on the work [@sekhar.rahul](https://medium.com/@sekhar.rahul/creating-a-time-lapse-video-on-the-command-line-with-ffmpeg-1a7566caf877) has done but is a more condensed version.

cd my/timelapse/folder

## testing the timelapse quickly

ffmpeg -framerate 30 -pattern_type glob -i "*.jpg" -s:v 1920x1080 -c:v libx264 -crf 17 -pix_fmt yuv420p my-timelapse.mp4

## apng for website posting

ffmpeg -i my-timelapse.mp4 -plays 0 -vf "scale=1920:-1:flags=lanczos,split [a][b];[a] palettegen [p];[b][p] paletteuse" my-timelapse.apng

ffmpeg -i my-timelapse.mp4 -f apng -plays 0 -vf "fps=15,scale=1920:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" output_file.png

Note: **fps=15,scale=1920** can be changed depending on size. E.g. 20 fps and 1280 if the image is too large

## 4k output H264

ffmpeg -framerate 30 -pattern_type glob -i "*.jpg" -s:v 3840x2160 -c:v libx264 -profile:v high444 -pix_fmt yuv422p10 my-timelapse-libx264.mp4

## slow down the video if not many frames

ffmpeg -i input_video.mp4 -filter_complex "[0:v]setpts=2.0*PTS[v];[0:a]atempo=0.5[a]" -map "[v]" -map "[a]" output_video.mp4

ffmpeg -i my-timelapse.mp4 -filter_complex "[0:v]setpts=2.0*PTS[v]" -map "[v]" -map "[a]" output_video.mp4
ffmpeg -i my-timelapse.mp4 -filter:v "setpts=2.0\*PTS" output.mp4

## Add panning

## 60fps panning

ffmpeg -i my-timelapse-libx264.mp4 -filter:v "zoompan=z=1.1:d=1:x='px+0.5':y='ih/2-(ih/zoom/2)':s=3840x2160:fps=60" my-timelapse-libx264-panning60fps.mp4

ffmpeg -i sunset-bw/my-timelapse-libx264.mp4 -filter:v "zoompan=z=1.1:d=1:x='px+0.5':y='ih/2-(ih/zoom/2)':s=3840x2160:fps=20" sunset-bw/my-timelapse-libx264-panning20fps.mp4

## 4k output prores (highest quality)

ffmpeg -framerate 30 -pattern_type glob -i "timelapse01/*.jpg" -s:v 3840x2160 -c:v prores -profile:v 3 -pix_fmt yuv422p10 my-timelapse-prores.mov


## Create a timelapse slow (portrait orientation)

```
❯❯❯ cd '/Volumes/Kingston Sv/2023-03-bodo/FINAL/timelapse-line2'
❯❯❯ ffmpeg -framerate 30 -pattern_type glob -i "*.jpg" -s:v 2160x3840 -c:v libx264 -crf 17 -pix_fmt yuv420p my-timelapse-prores.mp4
❯❯❯ ffmpeg -i my-timelapse-prores.mp4 -filter:v "setpts=5.0\*PTS" timelapse-line-prores-final.mp4
```
