---
title: "Revisit Path Tracing"
layout: post
image: 2024-07-01-revisit-path-tracing/victor-li-render-final-msaa-result.jpg
---

It's been a long time since last time I was learning the theories and implementation of raytracing by following the amazing [Ray Tracing in One Weekend](http://in1weekend.blogspot.com/2016/01/ray-tracing-in-one-weekend.html). Recently I finally decided to revisit the topic, since I have read so many related books and articles and also have improved a lot on personal tech stack that necessary for its implementation.

In this and the following posts I would like to write down my new understanding of the topic with references to all those great materials - probably for future re-revisit. It is fascinating to be able to discover new perspectives and nuances everytime taking another dive into the field. :)

---

# PBRT
After the previous playful implementation based on *Ray Tracing in One Weekend*, I got my hands on the 
*de facto* PBR bible [Physically Based Rendering:From Theory To Implementation](https://pbr-book.org/3ed-2018/contents) and started grinding on it. It was by no means an easy read as it is about a full-feature extremely physically accurate software renderer, based on solid scientific derivation and implemented in C++ with lots of consideration on architecture and system design.

<img src="https://i.ebayimg.com/images/g/E-oAAOSw181mMYFR/s-l1600.jpg" width="300" style="display:block; margin:auto;">

I started by reading the chapters:
- [1.2 Photorealistic Rendering and the Ray-Tracing Algorithm](https://pbr-book.org/3ed-2018/Introduction/Photorealistic_Rendering_and_the_Ray-Tracing_Algorithm.html)
- [1.3 pbrt: System Overview](https://pbr-book.org/3ed-2018/Introduction/pbrt_System_Overview.html)

which is a great entry point of the complex system.

Then [5.4 Radiometry](https://pbr-book.org/3ed-2018/Color_and_Radiometry/Radiometry) is a great scientific read about the **lighting physics** behind *pbrt*. The following [5.5 Working with Radiometric Integrals](https://pbr-book.org/3ed-2018/Color_and_Radiometry/Working_with_Radiometric_Integrals.html) and [5.6 Surface Reflection](https://pbr-book.org/3ed-2018/Color_and_Radiometry/Surface_Reflection.html) are also important by touching the key details of future implementation.

Next I jumpped to [8 Reflection Models](https://pbr-book.org/3ed-2018/Reflection_Models.html) which dives deeper on **BRDF** (and BTDF; and of course BRDF + BTDF => BSDF).

Finally I reached at the main course of **path tracing** implementation at [
14 Light Transport I: Surface Reflection](https://pbr-book.org/3ed-2018/Reflection_Models.html).

And of course, while reading the chaper above, I kept bumping into implementation details related to **Monte Carlo Integration**, which makes it very difficult to proceed instead I have to catch up at [13 Monte Carlo Integration
](https://pbr-book.org/3ed-2018/Monte_Carlo_Integration).

Similarly, lots of times I was pushed to pause and catch up at [7 Sampling and Reconstruction](https://pbr-book.org/3ed-2018/Sampling_and_Reconstruction.html) to brush up my **probability and statistics** theories(honestly these are the fields I sufferred the most back in university...).

Obviously, *pbrt* is not the best book to quickly guide me to produce path tracing pretty images, like last time, but it was a great experience for me to peek under the hood to see how much knowledge a simple path tracing demo is built upon - which is awe-inspiring, and also to form a clear structure in my mind of all the topics I need to further look into.

<img src="{{ site.url }}/images\2024-07-01-revisit-path-tracing\pbrt-chapters.png" width="300" style="display:block; margin:auto;">

The [pbrt source code](https://github.com/mmp/pbrt-v4) is very complex to dig through. However, [1.3.5 An Integrator for **Whitted Ray Tracing**](https://pbr-book.org/3ed-2018/Introduction/pbrt_System_Overview#AnIntegratorforWhittedRayTracing) offers a great minimal viable implementation for a quick path tracing demo. And while reading this part I followed this article [Overview of the Ray-Tracing Rendering Technique](https://www.scratchapixel.com/lessons/3d-basic-rendering/ray-tracing-overview/light-transport-ray-tracing-whitted.html
) on *scratchapixel.com* and its source code to learn more.

---
# smallpt
Another must visited place while I was doing my research was the [smallpt demo](https://www.kevinbeason.com/smallpt/) aka *small path tracer*.

> smallpt is a global illumination renderer. It is 99 lines of C++, is open source, and renders the above scene using unbiased Monte Carlo path tracing.

The [Presentation slides](https://docs.google.com/open?id=0B8g97JkuSSBwUENiWTJXeGtTOHFmSm51UC01YWtCZw) is very helpful for following the code.

I got the code built and run and produced some nice images. It is an amazing learning project as it offered a small code base instead of the whole source of pbrt. But figuring out the code is chanllenging as it exposed lots of missing areas of my knowledge base, for example:

- Russian Roulette
- Sampling Sphere light by Solid Angle
- Shadow Ray, etc.

There are also many cool path tracing **shadertoys** I was exploring, which helped me on trying to understand smallpt:

<iframe width="640" height="360" frameborder="0" src="https://www.shadertoy.com/embed/MclcR2?gui=true&t=10&paused=true&muted=true" allowfullscreen style="display:block; margin:auto;"></iframe>

# Classes - TU Wien
All the peripheral reading and researching was pushing me to want to learn more - maybe through a different medium, instead of staying on grinding the book or guessing how the source code just magically works.

I bumpped into this open courses [Rendering (186.101, 2021S)](https://youtube.com/playlist?list=PLmIqTlJ6KsE2yXzeq02hqCDpOdtj6n6A9&si=0UzTvrBhRnMOKXVr) which is such a gold mine. Not suprised to find out they are exactly using *pbrt* as their main teaching book. Important course are:

- [Rendering Lecture 01 - Light](https://www.youtube.com/watch?v=QgzqCLXX1OQ&list=PLmIqTlJ6KsE2yXzeq02hqCDpOdtj6n6A9&index=2&t=7s&pp=iAQB)
- [Rendering Lecture 02 - Monte Carlo](https://www.youtube.com/watch?v=_56eYqYYO6I&list=PLmIqTlJ6KsE2yXzeq02hqCDpOdtj6n6A9&index=3&t=1356s&pp=iAQB)
- [Rendering Lecture 03 - The Rendering Equation](https://www.youtube.com/watch?v=RBqQKGbxrsY&list=PLmIqTlJ6KsE2yXzeq02hqCDpOdtj6n6A9&index=4&pp=iAQB)
- [Rendering Lecture 04 - Path Tracing Basics](https://www.youtube.com/watch?v=w36xgaGQYAY&list=PLmIqTlJ6KsE2yXzeq02hqCDpOdtj6n6A9&index=5&pp=iAQB)

The course is really well structured and clear demonstrated, helped me comb through what I have read in the *pbrt* book and achieved better understanding.

## Eric Veach's Paper
PhD thesis by Eric Veach [Robust Monte Carlo Methods for Light Transport Simulation
](https://graphics.stanford.edu/papers/veach_thesis/) is mentioned during the course. After scanning it I realize the course is heavily based on it, especially the demonstration on the 3 **Formulation of Light Transport Equations**. Leaving a reference here for future reading as it is such a dense material.

# Intel Path-Tracing Workshop
I paused my study for a while until I came across this material:

- [Path-Tracing Workshop Part 1: Write a Ray Tracer](https://www.intel.com/content/www/us/en/developer/videos/path-tracing-workshop-part-1.html#gs.c74n6i)
- [Path-Tracing Workshop Part 2: Write a Path Tracer](https://www.intel.com/content/www/us/en/developer/videos/path-tracing-workshop-part-2.html#gs.c74p8h)


This workshop shows how to:

- Implement ray tracing in software on the GPU.
- Use GLSL, Shadertoy, camera models, ray-triangle intersection tests, and ray-mesh intersection tests.
- Build on your ray tracer and implement a path tracer that renders a scene with full global illumination.
- Learn about fundamental concepts in physically based rendering such as global illumination, radiance, the rendering equation, Monte Carlo integration, and path tracing.
- Implement the Monte Carlo integration, and use it to compute direct illumination.
- Write your path tracer.

More about the workshop from the author can be found on his [blog](https://momentsingraphics.de/PathTracingWorkshop.html).

Course project result on shadertoy:

<iframe width="640" height="360" frameborder="0" src="https://www.shadertoy.com/embed/MflczB?gui=true&t=10&paused=true&muted=true" allowfullscreen style="display:block; margin:auto;"></iframe>

# Side Quest - Real-time Rendering
At the meantime, I had another personal learning project going on to brush up my **OpenGL** knowledge and probably write a real-time renderer.

The current result has been pretty underwhelming - just **a model viewer** with basic lighting types support, done in **C++**:

<img src="{{ site.url }}/images\2024-07-01-revisit-path-tracing\opengl1.png" width="100%" style="display:block; margin:auto;">

- directional, point and spot light
- *assimp* for model loading
- *stb_image* for texture loading

<img src="{{ site.url }}/images\2024-07-01-revisit-path-tracing\opengl2.gif" width="400" height="400" style="display:block; margin:auto;">

Later with extra digressing, I reimplemented the renderer in **Python** for the sake of playing with [PyOpenGL](https://pyopengl.sourceforge.net/) and [Glumpy](https://glumpy.github.io/) libraries, which support **PBR shading models**:

<iframe src="https://player.vimeo.com/video/382872589?h=098e80235c" width="100%"  height="400" frameborder="0" allowfullscreen style="display:block; margin:auto;" ></iframe>

*^ I'm planning to write more posts about those real-time renderer projects, just as experiment notes...*

But the main point is all those tests made me wonder if I can connect both sides together - **a real-time demo done in OpenGL, but displaying a progressive path tracing result?**

It would be a great system since - most of the path tracing materials I was following are purely CPU based with an end result being a staitc image (PPM file). It is a slow process and it is by no means interactive.

However, the shadertoy path tracing demos I referred above are very interactive as it is implementing bare bone path tracing in GLSL hence the computation is on GPU. Technically, I can have my real-time renderer to render a quad that fill the entire window and have a fragment shader to render path tracing, and possibly have the demo taking user input to have interactivity.

This is my current idea of my next project. :)

# New inspiration...
It is lucky to land at
[yumcyawiz's blog - OpenGL3.3でGPUパストレーサーを実装する](https://blog.teastat.uk/post/2020/12/implementing-gpu-path-tracer-with-open-gl-3-3/) to find out someone achieved exactly the same goal!

<iframe width="100%"  height="400" src="https://www.youtube.com/embed/-dmQk2q3FTo?si=Kgzeyq-2eKlO2gQx" frameborder="0" allowfullscreen style="display:block; margin:auto;"></iframe>

His project [glsl330-cornellbox](https://github.com/yumcyaWiz/glsl330-cornellbox) is an amazing example to look into, with even extra effort on integrating **Imgui** for more interactivity. Even more, his project is based on [GLSL-PathTracer](https://github.com/knightcrawler25/GLSL-PathTracer) which is offering more interactive features!

Here is the result of building and running the demo:

<img src="{{ site.url }}/images\2024-07-01-revisit-path-tracing\cornellbox.jpg" style="display:block; margin:auto;">

Apart from that his another project [Luminox](https://github.com/yumcyaWiz/Luminox) is a minimal typical CPU path tracer and offers clean implementation. 

<!-- 
https://www.scratchapixel.com/lessons/mathematics-physics-for-computer-graphics/geometry/spherical-coordinates-and-trigonometric-functions.html

Simplifying Calculations for Trigonometric Ratios

template<typename T> inline T cosTheta(const Vec3<T> &w) { return w[2]; } -->

In the following posts I will put some of my study notes that I mentioned above, thanks to all those amazing resources I discovered online.

END

















