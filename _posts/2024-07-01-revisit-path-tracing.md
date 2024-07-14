---
title: "Revisit Path Tracing"
layout: post
image: 2024-07-01-revisit-path-tracing\victor-li-render-final-msaa-result.jpg
---

<img src="{{ site.url }}/images\2024-07-01-revisit-path-tracing\victor-li-render-final-msaa-result.jpg" style="display:block; margin:auto;">

It's been a long time since last time I was learning the theories and implementation of raytracing by following the amazing [Ray Tracing in One Weekend](http://in1weekend.blogspot.com/2016/01/ray-tracing-in-one-weekend.html). Recently I finally decided to revisit the topic, since I have read so many related books and articles and also have improved a lot on various technical tools necessary for its implementation.

In this and the following posts I would like to write down my new understanding of the topic with references to all those great materials - probably for future re-revisit. It is fascinating to be able to discover new perspectives and nuances everytime taking another dive into the field. :)

---

# PBRT
After the previous playful implementation based on *Ray Tracing in One Weekend*, I got my hands on the de facto PBR bible [Physically Based Rendering:From Theory To Implementation](https://pbr-book.org/3ed-2018/contents) and started grinding on it. It was by no means an easy read as it is about a full-feature extremely physically accurate renderer, based on solid scientific derivation and implemented with C++ with lots of architectual and system design consideration.

I started by reading the chapters:
- [1.2 Photorealistic Rendering and the Ray-Tracing Algorithm](https://pbr-book.org/3ed-2018/Introduction/Photorealistic_Rendering_and_the_Ray-Tracing_Algorithm.html)
- [1.3 pbrt: System Overview](https://pbr-book.org/3ed-2018/Introduction/pbrt_System_Overview.html)

which are great entry point of the system.

Then [5.4 Radiometry](https://pbr-book.org/3ed-2018/Color_and_Radiometry/Radiometry) a great scientific read about the physics base of pbrt. The following [5.5 Working with Radiometric Integrals](https://pbr-book.org/3ed-2018/Color_and_Radiometry/Working_with_Radiometric_Integrals.html) and [5.6 Surface Reflection](https://pbr-book.org/3ed-2018/Color_and_Radiometry/Surface_Reflection.html) are also important on touching the key details of future implementation.

Next was jumping to [8 Reflection Models](https://pbr-book.org/3ed-2018/Reflection_Models.html) which dived deeper on BRDF (and BTDF, and of course BRDF + BTDF => BSDF).

Finally I reached at the meat and bread of path tracing implementation at [
14 Light Transport I: Surface Reflection](https://pbr-book.org/3ed-2018/Reflection_Models.html)

And of course, while reading the chaper above, I kept bumping into implementation details related to Monte Carlo, which makes it very difficult to proceed instead I have to catch up at [13 Monte Carlo Integration
](https://pbr-book.org/3ed-2018/Monte_Carlo_Integration).

Similarly, lot of time I was pushed to pause and catch up at [7 Sampling and Reconstruction](https://pbr-book.org/3ed-2018/Sampling_and_Reconstruction.html) to brush up my probability and statistic theories.(honestly these are the fields I sufferred the most back in university...)

TBC

















