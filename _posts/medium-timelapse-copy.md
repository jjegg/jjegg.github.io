A post on how to create a timelapse video with ffmpeg. This was enspirted by [![Rahul Sekhar](https://miro.medium.com/v2/resize:fill:96:96/0*hRdtGMTFoDbj6WGd.).

```
medium-to-markdown@0.0.3 convert
node index.js https://medium.com/@sekhar.rahul/creating-a-time-lapse-video-on-the-command-line-with-ffmpeg-1a7566caf877
```

[![Rahul Sekhar](https://miro.medium.com/v2/resize:fill:96:96/0*hRdtGMTFoDbj6WGd.)

](https://medium.com/@sekhar.rahul?source=post_page-----1a7566caf877--------------------------------)[Rahul Sekhar](https://medium.com/@sekhar.rahul?source=post_page-----1a7566caf877--------------------------------)Follow

Jul 12, 2020

·8 min read

Creating a Time-Lapse Video Through the Command-Line (Using FFmpeg)
===================================================================

I tried creating a time-lapse video without using any video editing applications and would like to share my learnings. I found FFmpeg to be powerful — it can crop, trim, join videos, fade, add audio and even add moving zoom and pan effects.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

First things first, here’s one of the videos I created using FFmpeg:
--------------------------------------------------------------------

Time-lapse has fascinated me since I watched [Terje Sørgjerd’s videos](https://vimeo.com/terjes) many, many years ago. I’ve been going camping a lot this summer, and I’m discovering just how spectacular British Columbia is. So it seemed like a good opportunity to experiment with time-lapse myself. I bought a cheap, second hand GoPro Hero 3+ Silver and a portable tripod to get started, and took them along on a trip to [Manning Park](http://bcparks.ca/explore/parkpgs/ecmanning/).

Trying to find the right application to put the video together quickly got frustrating. Free software was limited and didn’t have much control, and I’m not quite ready to invest money into more professional software yet. But once I discovered how powerful the open-source command-line program [FFmpeg](https://ffmpeg.org/) is, I decided to give it a try.

Before I dive in, I should say that putting together a video on the command-line is quick, and efficient and gives you a lot of fine-grained control when you’re doing simple things (a straight time-lapse, joining together videos with simple transitions, cropping, adding an audio track in). **But when things start to get more intricate it can be painful to work without the immediate feedback a video editing application gives you** — for example when you’re trying making timing changes to sync up to your audio track, or zooming/panning.

First step — a quick and dirty time-lapse
=========================================

I’m going to assume you have FFmpeg set up (if not, [here’s some info on how to install it](https://github.com/adaptlearning/adapt_authoring/wiki/Installing-FFmpeg)) and know how to run it on terminal. I’ll also assume your sequence of images is ready in a single folder and are named such that they’re displayed in the order that they should be rendered. Let’s run a quick command to convert them into a video.

```
ffmpeg -framerate 30 -pattern\_type glob -i "folder-with-photos/\*.JPG" -s:v 1440x1080 -c:v libx264 -crf 17 -pix\_fmt yuv420p my-timelapse.mp4
```

The final parameter is the filename of the output video. Here’s what the other parameters mean:

*   **framerate:** the number of images to render per second in the video
*   **pattern\_type:** we set this to glob to tell ffmpeg to use all the images that match the pattern in the following parameter.
*   **i:** a pattern that matches all the input photos in your timelapse. Note that this, as with most other things on a UNIX command line, is case sensitive.
*   **s:v:** The size of the output video. Ensure that the aspect ratio matches your photos to avoid skewing the images (we’ll talk about cropping later).
*   **c:v:** The output video codec (here, H264). More on this later.
*   **crf:** A parameter specific to the H264 codec that determines the quality/compression. More on this later.
*   **pix\_fmt:** this needs to be set to yuv420p to allow many players, such as Quicktime to play the video (the FFmpeg docs say \*dumb\* players, I’ll be more forgiving). More on this later.

This should create a quick time-lapse from your photos. I did this immediately with my raw images, before doing anything further, just to get a preview of the output. Play with the frame-rate to try out different time-lapse speeds. Ideally, your frame-rate should not be below 24 or the video will likely appear choppy.

_Note: There is some significance in the ordering of FFmpeg parameters. The parameters before_ `_-i_` _are all input options, and those after it are all output options._

_\[_[_More info on using an image sequence as an input._](https://en.wikibooks.org/wiki/FFMPEG_An_Intermediate_Guide/image_sequence)_\]_

Video codecs and formats
========================

**H264 and ProRes**
-------------------

The world of video codecs is vast, and I don’t have the knowledge to delve deep into it, so I’m just going to go briefly over my practical understanding.

The most commonly used codec on the web today is H264, which gives excellent quality video at amazingly small file sizes. This is what I would use for my final output, to be uploaded to an online platform. More details on encoding H264 with FFmpeg can be found [here](https://trac.ffmpeg.org/wiki/Encode/H.264). The most important parameter for encoding H264 is probably `crf`, which determines the compression level and quality of the output. The range is 0–51 and a lower number means higher quality and higher file-size. I use 17 when I want visually nearly lossless video, with manageable file-sizes.

The other codec I use is ProRes. This is a high quality codec (and it generates very large file-sizes) intended for usage during video editing rather than for playback. Unlike H264, each frame in a ProRes video is independent of the other frames — while this means much less compression, it also means better performance inspecting individual frames in video editing software. More details on ProRes with FFmpeg and a comparison of some other codecs can be found [here](https://trac.ffmpeg.org/wiki/Encode/VFX). Here’s the same command above changed to output ProRes:

```
ffmpeg -framerate 30 -pattern\_type glob -i "folder-with-photos/\*.JPG" -s:v 1440x1080 -c:v prores -profile:v 3 -pix\_fmt yuv422p10 my-timelapse.mov
```

Workflow and usages of codecs
-----------------------------

I found that I have a few different use cases in my workflow:

*   **Preview video:** Initially, I want my video to be processed quickly so I can play around with different commands and settings and fine-tune my processing plan until I have the final video just how I want it. I don’t care much about the quality here, just the processing speed. For this, I use the default H264 settings, without specifying `crf`, and I use low resolutions like 800x600 or 800x450.
*   **Intermediate video:** Once I have my processing plan ready, I might still need to process the video in a series of steps rather than a single command. Each step generates an intermediate video. Any quality loss is magnified by subsequent encodings so it’s important to use a nearly lossless output. I use ProRes for this, but I daresay H264 with a very low CRF (near 0) would probably work as well.
*   **Storage video:** Once I finish processing my video, I might want store a near lossless version of it in case I ever need to process it further for use in a different context (say as part of a larger video). I would use ProRes for this, since it should be more performant to work with in video editing software than H264.
*   **Publishing video:** The storage version of the final video is large (many gigabytes for less than a minute of video), so to upload and publish my video I compress it to H264 at CRF 17, which is still high quality but with a manageable file size.

pix\_fmt
--------

A quick note on `pix_fmt` (pixel format). Without going into too much detail ([see more here](https://en.wikipedia.org/wiki/Chroma_subsampling)), this determines how much colour information is stored in each pixel and how much is shared between adjacent pixels. 4:2:0 (or `yuv420)`, is extremely widely used and is generally the right choice for final playback purposes. 4:2:2 or 4:4:4 store more colour information in each pixel and might appear slightly more accurate/sharper in colour. I use them only for intermediate/storage purposes since online platforms will generally encode to 4:2:0 in any case.

Processing the video
====================

FFmpeg is very flexible and there are endless things you can do with it. I aim to give you a few examples here based on what I did.

Note: I’ll omit the encoding parameters in each example for brevity, but you should set them or FFmpeg will use its defaults (unless you don’t care, say for a preview/test video).

Change speed
------------

You can speed up (or slow down a video) by using the `setpts` filter. To double the speed of the video:

```
ffmpeg -i input.mp4 -filter:v "setpts=0.5\*PTS" output.mp4
```

slow down
To increase the speed by 1.5x, use `setpts=0.667*PTS`, and so on. Note that this will discard frames form the video.

Crop
----

You can crop a video, choosing the size of the crop box and its left and top offset positions. To crop a video to 1920x1080, with the left offset of the cropping box at 100px and the top offset at 50px, use:

```
ffmpeg -i input.mp4 -filter:v "crop:1920:1080:100:50" output.mp4
```

Trim
----

You can trim a video, extracting a section of it. To take15.44 seconds of video starting at the 6.5 second mark:

```
ffmpeg -i input.mp4 -ss 6.5 -t 15.44 output.mp4
```

Joining multiple videos
-----------------------

To join together two videos of the same format, the simplest way is to create a file `mylist.txt` with the contents:

```
file 'path/to/video-1.mp4'  
file 'path/to/video-2.mp4'
```

Then run:

```
ffmpeg -f concat -safe 0 -i mylist.txt -c copy concat-video.mp4
```

_Note that this command does not re-encode the videos unlike the other commands above._ `_-c copy_` _tells it to directly copy them._

Transitions
-----------

To fade in a video at the beginning with a fade duration of 0.5s:

```
ffmpeg -i input.mp4 -filter:v "fade=t=in:st=0:d=0.5" output.mp4
```

To fade out a video at it’s end, first find it’s exact length:

```
ffprobe -show\_format input.mp4
```

You should see the duration listed. Let’s say the duration is 45.677s and I want a fade duration of 0.5s, then the fade will need to start at 45.177s, so the command will be:

```
ffmpeg -i input.mp4 -filter:v "fade=t=out:st=45.177:d=0.5" output.mp4
```

To crossfade two video or add other more elaborate transitions, [ffmpeg-concat](https://github.com/transitive-bullshit/ffmpeg-concat) might be a useful tool.

Combining filters
-----------------

Multiple video filters can be combined in one command, for example to crop and also add a fade in and fade out:

```
ffmpeg -i input.mp4 -filter:v "crop:1920:1080:100:50,fade=t=in:st=0:d=0.5,fade=t=out:st=45.177:d=0.5" output.mp4
```

Adding an audio track
---------------------

To add an audio track to a video:

```
ffmpeg -i input.mp4 -i my-audio.m4a -c copy -map 0:v:0 -map 1:a:0 -shortest output.mp4
```

Note that `shortest` sets the length of the output video to the shortest of the two inputs. You might want to first encode your audio to AAC or some other audio codec (see [here](https://trac.ffmpeg.org/wiki/Encode/AAC)). Audio fades can be applied very similarly to the video fades above:

```
ffmpeg -i input-with-audio.mp4 -filter:a "afade=t=in:st=0:d=0.5" output.mp4
```

Zooming/panning
---------------

Zooming or panning with FFmpeg gets a little more involved. For the details, the [docs](https://ffmpeg.org/ffmpeg-filters.html#zoompan) are probably your best source. To zoom or pan a timelapse, ideally the input resolution should be larger than my intended output resolution so we don’t lose quality on the zoom. Here’s an example of a pan effect:

```
ffmpeg -i input.mp4 -filter:v "zoompan=z=1.1:d=1:x='px+0.5':y='ih/2-(ih/zoom/2)':s=2560x1920:fps=30" output.mp4
```

*   `z=1.1`simply sets the zoom level to 1.1x so that we have space around to pan to.
*   `d=1` means that the effect lasts for a single frame for each input image. We always set `d` to 1 for a time-lapse, it’s only useful to set it higher for a slideshow or something like that.
*   `x='px+0.5'` sets the horizontal offset for each frame 0.5 pixels more than the previous value.
*   `y='ih/2-(ih/zoom/2)'` simply keeps our zoomed image section vertically centred.
*   `s=2560x1920` and`fps=30` set the output resolution and framerate.

Here’s an example of a zoom out effect:

```
ffmpeg -i input.mp4 -filter:v “zoompan=z='if(lte(in,1),1.2,max(pzoom-0.0001,1))’:d=1:x=’iw/2-(iw/zoom/2)’:y=’ih/2-(ih/zoom/2)’:s=2560x1920:fps=30" output.mp4
```

*   `z='if(lte(in,1),1.2,max(pzoom-0.0001,1))'` — let’s break this down. The `if` statement initialises the zoom to 1.2x (if the zoom is less than or equal to 1, which is the case for our first frame). After this, for every frame the zoom is evaluated to `pzoom-0.0001`, which is 0.0001 less than the zoom value for the previous frame. The `max` statement ensures that the zoom value stops reducing when it reaches 1.
*   The remaining params are the same as for the pan effect above. This time both the x and y params simply centre the zoomed box.

Reference
---------

The FFmpeg docs are excellent for more filters and more info: [https://ffmpeg.org/ffmpeg-filters.html](https://ffmpeg.org/ffmpeg-filters.html)

Fin
===

I hope you enjoy playing with FFmpeg. While it is powerful, as I said, there’s a limit to how efficient it is to process videos by command line rather than hands-on.

Let me know if you have any corrections for the article or any suggestions for a good, low-budget application that I can use the next time around!
