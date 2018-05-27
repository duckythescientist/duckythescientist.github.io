---
layout: post
title: SmoothLife Android -- Minimum Viable Product
tags: android smoothlife life gol conway
---

The second in a series of posts about making SmoothLife an Android Live Wallpaper.

![SmoothLife](https://raw.githubusercontent.com/duckythescientist/SmoothLife/master/img/smoothlife.gif)


# Android Development

I don't actually know Java. At least I don't think I know Java. I managed to write it anyway. I just met in the middle between C/C++ and Python and then let the autocomplete magic of Android Studio do the rest.

## Baby Steps

I found a few tutorials on Android live wallpapers, read through them to get the general ideas, and started with the basic framework.

* <https://www.faultinmycode.com/2018/05/android-live-wallpaper-tutorial.html>
* <http://androidsrc.net/android-live-wallpaper-tutorial/>
* <https://www.codeproject.com/Articles/1167401/Android-Live-Wallpaper-Tutorial>
* <http://www.vogella.com/tutorials/AndroidLiveWallpaper/article.html>

Next I re-implemented the kernel generation and state transition functions. Things were looking good so far, but I hadn't really tested much.

## Fourier Transforms

These are really important to the speed of the program. Without Fourier magic, the program runs in a slugging `O(n⁴)` time. With Fourier magic, that's reduced to `O(n²·log(n))`. I had also done some profiling of my Python implementation to show that even with the `O(n²·log(n))` version, the Fourier transforms were still a considerable portion of the processing time. This part had to be fast.

I had a few options:

1. Write a Fourier transform function in Java
2. Find a Java library that I could include
3. Find a native C/C++ library with bindings (or make bindings)

`1.` and `2.` would be slow, but `2.` would be the easiest. I started by trying to find a Java library, and as far as I could tell, there were slim pickin's. Apache Commons had a version, but it looked like only a 1D transform (unless I did some fanciness). Also, it had the limitation that the input array must be a power of 2 in length. This is a common requirement for FFT algorithms, but it was unfortunate for me because phone screens don't come in only power of 2 resolutions. 

Option `3.` was looking like the way to go. The library [FFTW](http://fftw.org/) library is pretty much the go-to library when it comes to native FFT. It's fast, and it's fully featured. But it's not Java. The downloads page linked to a couple Java wrappers, and I also found a couple older repositories that had ported FFTW to work with Android's `ndk-build` compiler suite.

As far as I can tell, when building things with Android Studio, you have to use a horrible mishmash of Gradle and CMake. I hate CMake because the extra abstraction layer from Gnu Make means it's impossible to debug, and every time I try to use Gradle, it's poorly documented and it has changed enough from the last time that I used it that none of the tutorials or references I can find are relevant anymore.

I really really wanted to find a solution that just worked. Enter [MVNRepository](https://mvnrepository.com/artifact/org.bytedeco.javacpp-presets/fftw/3.3.4-0.11) with a precompiled version of FFTW with `android-x86` and `android-arm` binaries. That was a tad out of date and didn't have enough different Android architectures for my tastes. Also, because of some other issues, I was thinking that the FFT wasn't working properly for some unknown reason. I eventually fixed that issue and found the [Bytedeco JavaCPP Presets Github](https://github.com/bytedeco/javacpp-presets) that has precompiled native libraries with Java bindings for a whole bunch of useful native libraries. 

My FFT solution was finally solved by shoving a few `.jar` files into the project's `./lib` directory.

I knew I now had fast FFTs, but I was still worried about the overhead of converting between Java datatypes and native datatypes. (Later profiling shows that it's not a huge issue.)

## Colormaps

I have strong opinions about colormaps. Read [this](https://cran.r-project.org/web/packages/viridis/vignettes/intro-to-viridis.html) and [this](https://bids.github.io/colormap/) if you, too, want strong opinions about colormaps.

Specifically, I really like the colormap Viridis. (Plasma is pretty nice too.) I wanted pretty colors, so I needed to get a colormap implementation in Java. I didn't find any great libraries for it, but I did find a [Java file in a project for circle detection in images](https://github.com/tinevez/CircleSkinner/blob/master/src/main/java/net/imagej/circleskinner/util/ColorMap.java) that looked like it'd do the trick. It had a couple other colormaps (including Jet and Parula) and Viridis.

Unfortunately it used a Java `Color` class that wasn't compatible with the Android `Color` class, so I made some modifications to convert everything to the `ARGB_8888` format that Android would be happy with.

OR SO I THOUGHT!!!

## Fourier Transforms Not Working

Getting a working FFT has been the hardest part of the project so far. There's a text padding issue that comes in second, but more on that later.

I ended up stuck on an issue for several hours. The style of FFT that best fit my usage was FFTW's `r2c` and `c2r` style DFTs. My spacial domain data was all real which meant that the Fourier domain complex data could be cut in (almost) half without losing any information. This transform is also faster that the complex to complex style. And because I'm dealing with real data half of the time instead of always complex data, it would mean a good bit less overhead when passing data between Java and native.

There is weirdness with the `r2c` and `c2r` transforms. It gets even weirder in 2D. Say you have an `n·m` array (`double[n][m]`) of real data. When you run a 2D `r2c` transform on it, you get an `n·(m/2 + 1)` array of _complex_ data. Complex numbers in this library are represented by an array of two doubles. This means the `r2c` output now has the format `double[n][m/2 + 1][2]`. This is all flattened arrays, so really it's of the format `double[n·(m/2 + 1)·2]`. For several hours, I was missing that last `2` in my arrays. I gave up and went to bed. Starting the next day, it was about 3 minutes until I found the error.

## Fourier Transforms Not Working 2

All of the rest of the basic code was in place, but my overall output wasn't looking right. Instead of debugging the whole system, I stepped back and started looking at internal states.

Doing `iFFT(FFT(data))` produced `data` properly, so my transforms were at least not too broken.

Next debugging step was try some basic kernel convolutions. For discussion here, I'm going to assume non-rotated kernels (middle of the kernel is in the middle of the array) and ignore all of the zero padding and just show 3x3 kernels. I'm also going to ignore the fact that kernels should technically be reversed in the convolution process.

First step: identity kernel

```
[[0, 0, 0],
 [0, 1, 0],
 [0, 0, 0]]
```

Convolution with this kernel will leave the input exactly the same. This worked.

Next step: shift kernel

```
[[0, 1, 0],
 [0, 0, 0],
 [0, 0, 0]]
```

Each time this kernel is convolved with the input, the output will be shifted down one row. This worked. I also tried a couple very similar ones.

Next step: Gaussian blur

```
[[1, 2, 1],
 [2, 4, 2],
 [1, 2, 1]] / 16.0
```

When applied, this kernel will blur the image. This didn't work. It would somewhat blur, but then I'd get speckles and bands of what looked like [ringing artifacts](https://en.wikipedia.org/wiki/Ringing_artifacts). That's a somewhat expected thing for image processing, but the fix is low pass filtering. A Gaussian blur is a low pass filter. It shouldn't be ringing.

Next thing I tried was just displaying the complex output of the Fourier transforms of known things. The Fourier transform of the following kernel (plus padding) has real values that ramp down from low to high frequencies. 

```
[[0, 0, 0],
 [0, 1, 0],
 [0, 1, 0]] / 2.0
```

The specifics don't matter. But showing the real values of the FFT of that kernel was a great way to show that the FFT was producing expected results. This is what it should look like vs. what it did look like:

![FFT cmap weirdness]({{ "/assets/bad_fft_cmap.png" | absolute_url }})

(Note, this isn't actually from the phone. I'm just mostly matching the visual with some Python.)

It looked close. The general features were there, but there was a lot of what looked like noise. I tried low pass filtering and some windowing. I even wrote a test function that used a symmetric (complex to complex) fft function from FFTW to see if it was some `r2c` and `c2r` weirdness. Those had the same issues.

Eventually, I had the good idea to just do a straight conversion to grayscale instead of going through the colormap library. Turns out that the problem was with the colormap.

Internally in the colormap library, it has a list of colors and a list of "alphas". The alphas are the values between 0 and 1 that correspond to each color in the list. For a lookup operation, there probably won't be the exact color in the list, so it'll find the two colors in between what the output should be and do a linear interpolation between the two. I was interpolating between colors. I should have been breaking the color into the RGB channels, interpolating between each one of those, and recombining into a final color at the end. Doing that fixed my FFT issue. 


----------------------

It was at this point that I had all of the basic functionality. It was buggy and slow as balls, but it worked. 