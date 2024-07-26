---
title: "Study Notes: GLSL CornellBox Breakdown - Part 2"
type: article
layout: post

---
This post is the second part of my study notes on breaking down yumcyawiz's project [glsl330-cornellbox](https://github.com/yumcyaWiz/glsl330-cornellbox). This time will focus on the shader side of the renderer and look into the real implementation of path tracing in GLSL.

* This will become a table of contents (this text will be scrapped).
{:toc}

---
# Overview

The author breaks the shaders into multiple small bite pieces and have them included into the main shader. This makes the implementation very structured and it is even closely assemble the architecture of PBRT, which helped me a lot to have a bird's eyes view of the system.

# pt.frag
This is the main shader contains function `computeRadiance(in Ray ray_in)` that calculate the path tracing result.