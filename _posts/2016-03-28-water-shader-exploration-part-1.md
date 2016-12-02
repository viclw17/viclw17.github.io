---
layout: post
title: Water Shader Exploration Part-1
description: "Water Shader Exploration Part-1"
modified: 2016-03-28
tags: [Unity Shader]
image:
  feature: water.jpg
  credit:
  creditlink:
---
## Intro

First try of water effect in Unity.

One of my current VR projects needs a nice and light water shader. I have been working on it for a while and keep iterating the the overall effect. Here is the water effect I have achieved so far, however it's still far away from perfect simulation:

<img src="{{ site.url }}/images/water0.gif" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Final work w/o transparency.</figcaption>

Water effect has been always a complicated topic in shading especially for runtime scenario. As the most usual material in our daily life, water has lots of visual features that we are very familiar with. However, all these features also got lots of physics and lighting formulas involved which are interacting with each other.

I started with thinking about the most typical physical and lighting features that came into mind and got my hand dirty immediately to achieve as much as I can. Here is my **checklist**:

* Blue-ish color.
* Waving surface.
  + Using script to deform a plane mesh, **OR**
  + Manipulating vertex in vertex shader.
* Ripple.
  + Texture & Normal Map
  + UV Scrolling
* Reflection.
  + Specular lighting model
  + Cubemap
* Transparency.
* Refraction. (no idea...)

I firstly wrote a script to move the vertex of a plane mesh according to sine wave, and added [Perlin Noise](http://docs.unity3d.com/ScriptReference/Mathf.PerlinNoise.html) function to add some randomness. Then I started to write my water shader to turn this waving plane into a water like surface with all the interesting lighting attributes that I can think about. In Part-2 I will go through my exploration in details.

<!-- <img src="{{ site.url }}/images/water1.gif" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Final work w/o transparency.</figcaption> -->

<img src="{{ site.url }}/images/water2.gif" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Final work w/ transparency and faint tint and BUNNY!</figcaption>

The current water effect is enough for the scope of my project and it's light to implement and has decent visual features. However, this is just a very very high-level implement using Unity and for simple environment. Also, there are a lot of features that are very important in water, like refraction, that I haven't implemented. I will keep working on my exploration on this topic in the future.

P.S. My ultimate goal about water simulation could be **Low-Level GPU-Based Volumetric Fluid Simulation**. Hopefully I'm able to give a shot one day. Video here just for admiring and future reference. :)

<!-- 
<iframe src="https://player.vimeo.com/video/87050516?autoplay=1&loop=1&title=0&byline=0&portrait=0" width="500" height="281" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/87050516">PIC/FLIP Simulator Dam Break Test- Final Render</a> from <a href="https://vimeo.com/user3522674">Yining Karl Li</a> on <a href="https://vimeo.com">Vimeo</a>.</p>
-->

(TBC)
