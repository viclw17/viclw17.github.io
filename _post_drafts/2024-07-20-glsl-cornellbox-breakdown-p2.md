---
title: "Study Notes: GLSL CornellBox Breakdown - Part 2"
type: article
layout: post

---
This post is the second part of my study notes on breaking down yumcyawiz's project [glsl330-cornellbox](https://github.com/yumcyaWiz/glsl330-cornellbox). This time I will focus on the shader side of the renderer and look into the real implementation of path tracing in GLSL.

* This will become a table of contents (this text will be scrapped).
{:toc}

---
# Overview

The author breaks the shaders into multiple small bite pieces and includes them in the main shader file. This makes the shader code very structured and it even closely resembles the architecture of PBRT, which helped me a lot to have a bird's eyes view over the system.

# pt.frag
This is the main shader file with 'main()' function and contains the function `computeRadiance(in Ray ray_in)` that calculates the path tracing result. 