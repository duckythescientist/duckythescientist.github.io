---
layout: post
title: SmoothLife Android -- Optimization
tags: android smoothlife life gol conway optimization
---

The third in a series of posts about making SmoothLife an Android Live Wallpaper.

![SmoothLife](https://raw.githubusercontent.com/duckythescientist/SmoothLife/master/img/smoothlife.gif)

# Optimization

At this point I had a working wallpaper. The frame rate was miserable, and it was taking up more of my phone's battery than the rest of everything else _combined_.

## Obvious First Step -- Scaling

Instead of computing SmoothLife at full resolution, compute at a partial resolution and then upscale. Turns out that with the cell `INNER_RADIUS` at 7, everything still looks good with scale factors around 4 to 6. This made things a bunch faster, but there was still room for improvement. (This has only been tested on phones up to full HD. Higher resolutions will likely need higher scale factors.)

After this, I wasn't sure where exactly would be best to improve, so it was time for profiling.

## Profiling

Android Studio has a pretty reasonable built-in sampling profiler. The UI is still a tad clunky, but it's useful for overall things.

### Colormap Binary Search

The colormap code included a binary search to find the two colors to interpolate between. In theory, this is good because it allows for colormaps to be built with reference points that aren't evenly distributed, but the binary search was taking way too long. All of the color maps that previously existed in the file (and the grayscale map that I added) had evenly spaced reference points. The binary search could be replaced by some simple math and an array lookup. `int i = (int)(val * (nColors - 1));` then look up between `colors[i]` and `colors[i + 1]`. That saved a handfull of milliseconds per frame.

### State Transition Function

From some profiling of the Python version, I knew that this was going to be an issue. The division and especially the exponentials in the state transition function are costly math operations. There are 3 exponentials and 6 divisions per grid element. Lookup tables to the recsue.

I pre-compute a 512x512 element lookup table for the possible `n` and `m` inputs. Fewer elements than that and the image starts to get a little quantized. More than that is unnecessary. The computation of the table takes a third to a half second or so, but it's only called when the wallpaper surface is first created since it's independent of resolution.

It's a little less than ideal because it's quantizing something that is supposed to be continuous, but that's happening anyway since we are doing everything on a grid.

### Threading

There were a few things that could be done at the same time.

The two inverse Fourier transforms can be parallelized, but there's a good amount of overhead in the thread startup and join. On my Pixel 2 with a Scale of 4, I'm not sure that I'm getting much speedup, so I haven't yet decided if I want to single or multi thread this part.

The operation of locking the canvas for drawing is surprisingly time intensive. To get around this, I span the `step` function in a separate thread then request the canvas, draw, and post. This was working, but I was getting artifacts like screen tearing because the bitmap was being updated in the middle of the drawing process. I fixed that with double buffering. I now have two different bitmaps. I update one while I draw the other to the screen.

On my Nexus 5 AVD (emulator) and a Scale of 4, the Android canvas operation were now taking up all of the time. I can't go faster unless I change the way Android standardly does live wallpapers. It's not quite that fast in real life, but it's at least much better. Pixel 2 draws at about 10fps with a Scale of 4 and 20fps with a scale of 6. 

Battery drain is now much more reasonable, and frame rates are fast enough that adding an extra frame delay is necessary for SmoothLife to look good (in my opinion).