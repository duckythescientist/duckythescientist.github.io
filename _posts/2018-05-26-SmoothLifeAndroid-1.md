---
layout: post
title: SmoothLife Android -- Background
tags: android smoothlife life gol conway
---

The first in a series of posts about making SmoothLife an Android Live Wallpaper.

![SmoothLife](https://raw.githubusercontent.com/duckythescientist/SmoothLife/master/img/smoothlife.gif)

I have already written [a version of SmoothLife in Python](https://github.com/duckythescientist/SmoothLife) that is well documented and explained, so I'd recommend starting there.

# Motivation and Background

## Why?

Conway's Game of Life (GoL) is a beautiful example of a simple system with complex emergent behavior, and it's a program and concept that I keep coming back to year after year. I wrote an [obfuscated version in 9 lines of C](https://github.com/duckythescientist/obfuscatedLife) 5 years ago. Before that, I had implemented it a few different times on a few different devices going all the way back to my TI-84+ in high school. TI-Basic on that calculator is pretty much how I first learned programming.

I also love visualizations of math things in 2D. The Mandelbrot Set is another of my favorite programs. Yet again, complex behavior from a simple system. I don't do overly fancy visualizations (e.g. no three.js). Instead, I really like when I can just create a large matrix and blit it onto a screen.

Probably around 4-6 years ago, I saw a Reddit post that linked to [a video of SmoothLife](https://www.youtube.com/watch?v=KJe9H6qS82I). I quickly skimmed the accompanying research paper, but it was a tad over my head at the time. I saved it for later. Recently, while cleaning through the cruft of saved Reddit posts, I found it again and also found a [great post](https://0fps.net/2012/11/19/conways-game-of-life-for-curved-surfaces-part-1/) that went over the math and implementation details for SmoothLife. It also linked a version written in JavaScript.

The algorithm itself has some really nifty ideas that drew me in. I like math -- specifically the engineering type of math. I don't like it when the math stops having numbers. Among other things, I have an electrical engineering background that had a good bit of focus on signals. Fourier transforms are incredibly important for that. I actually managed to fall in love with them before I had any classes that dealt with them. They are still magic to me, but I'm at least to the point that I know where and how to apply them. I also know how little I still know about them.

SmoothLife managed to have aspects of so many things that I really liked, so I had to work on it.

The code that I had seen wasn't written in a way to be quickly understandable, so I decided I'd do it better.

## Python

I love Python. It's the shortest gap between idea and working code (for me) that I've found yet. I was also looking for an excuse to get better at Numpy. Mostly using the [JavaScript version](https://jsfiddle.net/mikola/aj2vq/) as a reference, I re-implemented everything in Python while taking the time to figure out what was going on and why things were the way they were. The first draft took about a day, and I fixed/improved bits and pieces over the next couple weeks. I'm funemployed right now, so I had the time.

## Live Wallpaper

For those of you who aren't overly familiar with Android, there's a feature where you can have an application that provides moving (and interactive) animations for the wallpaper. I loved the look of my Python SmoothLife, and I thought it would be really neat to get it running as an Android live wallpaper.

I had only done minimal Android development in the past. Also, the Python SmoothLife running on my desktop was only getting 4ish frames per second when running at 1080x1920. My other option would be OpenGL or some other GPU implementation, but that gets nasty. Lack of Android experience and the knowledge that it'd be horribly slow and a huge battery drain made me put the idea into the back of my mind for a while.

I needed to pick a project to work on this week, so I decided to go for it knowing that it'd probably be a huge failure. Even if I got it working, the frame rate would be so miserable and the hit on battery life would be so bad that it wouldn't be worth having. That all notwithstanding, I decided to try it. I'm glad I did.


