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
Problems:
1. We can map objects with images to add visual complexity to their appearance.
2. Computers had a very limited memory and images used for texturing would not easily fit in RAM.
3. However using a RNG (random number generator) to add variation to the appearance of a 3D object isn't sufficient.
4. Random patterns we can observe in nature are usually smooth.
5. In other words local changes are gradual, while global changes can be large.

white noise :(

The conclusion of this experiment is that to create **a smooth random pattern**, we need to assign random values at fixed position on a grid which we call a lattice (in our example the grid corresponds to the pixel locations of our 10x10 input image) using a RNG, and blur these values using something equivalent to a **gaussian blur** (a smoothing function to blur these random values). In the next chapter we will show you how this can be implemented. **But for now all you need to remember is that noise (within the context of computer graphics) is a function that blurs random values generated on a grid (which we often refer to as a lattice).**

# Properties of an Ideal Noise
1. Noise is **pseudo-random** and this is probably its main property. We say that the function is invariant: it doesn't change under transformations).
2. The noise function **always returns a float no matter what the dimension of the input value is**. The dimension of the point is given to the name of the noise. A 1D, 2D, 3D noise functions are functions taking 1D, 2D and 3D points as input parameters. There is even a 4D noise which takes a 3D point as input parameter and an additional float value which is used to animate the noise pattern over time (time varying solid texture). In mathematical terms we say that the noise function is a mapping from Rn to R (where n is the dimension of the value passed to the noise function). It takes a n-dimensional point with real coordinates as input and returns a float. 1D noise is used for animating objects, 2D and 3D noise are used for texturing objects. 3D noise is particularly useful for modulating the density of volumes.
3. Noise is **band limited**. Remember that noise is mainly a function, which you can see as a signal (if you plot the function you get a curve which is your signal). In signal processing it's possible to take a signal and convert it from spatial domain to frequency domain. This operation gives a result from which it is possible to see the different frequencies that a signal is made of. All you need to remember is that the noise function is potentially made of multiple frequencies (low frequencies account for large scale changes, high frequencies account for small changes). But one of these frequencies dominates all the others. And this one frequency defines both the visual and frequency (if you look at your signal in frequency space) appearance/characteristic of your noise function. Why should we care about the frequency of a noise function? When the noise is small in frame (imaging an object textured with noise far away from the camera) it becomes white noise again which is a cause of what we call in our jargon, **aliasing**.
4. Aliasing is related to the topic of **sampling** which is a very large and very important topic in computer graphics.
5. We mentioned that noise uses **a smooth function to blur random values** generated at lattice points. Functions in mathematics have properties. Two of them are of particular interest in the context of this lesson: **continuity** and **differentiability**. If the function is not continuous, computing this derivative is not possible (a function can also be continuous but not differentiable everywhere, as shown in figure 5 on the right). For reasons we will be explaining later, computing derivatives of the noise function will be needed in some places and it is **best to choose a smooth function which is both continuous and differentiable**. The original implementation of the noise function by Ken Perlin used a function which wasn't continuous and he suggested another one a few years later to correct this problem.
6. Ideally what you want is an invisible transition from tile to tile so you can cover an infinitely large area without ever seeing a seam. In CG when a 2D texture is seamless, it is said to be **periodic** in both direction (x and y). The word tileable is also sometimes used but is confusing. Any texture is tileable only it might not be seamless. Ideally your noise function should be designed so that the pattern is periodic.

https://www.scratchapixel.com/lessons/procedural-generation-virtual-worlds/procedural-patterns-noise-part-1/creating-simple-1D-noise
more notes...

https://www.scratchapixel.com/lessons/procedural-generation-virtual-worlds/procedural-patterns-noise-part-1/creating-simple-2D-noise
