---
title: Noise Study (GLSL)
date: 2019-01-15
tags:
- Computer Graphics
- GLSL
- Math
- Noise
---
Keywords:
value noise,
procedural pattern generation,
deterministic,
random number generator, RNG,
periodic, continuity, differentiability,
sampling, aliasing, white noise, solid textures,
permutation, hash table.

---
https://www.scratchapixel.com/lessons/procedural-generation-virtual-worlds/procedural-patterns-noise-part-1/introduction
We can map objects with images to add visual complexity to their appearance.
computers had a very limited memory and images used for texturing would not easily fit in RAM.
In other words local changes are gradual, while global changes can be large.

white noise

The conclusion of this experiment is that to create a smooth random pattern, we need to assign random values at fixed position on a grid which we call a lattice (in our example the grid corresponds to the pixel locations of our 10x10 input image) using a RNG, and blur these values using something equivalent to a gaussian blur (a smoothing function to blur these random values). In the next chapter we will show you how this can be implemented. But for now all you need to remember is that noise (within the context of computer graphics) is a function that blurs random values generated on a grid (which we often refer to as a lattice).
