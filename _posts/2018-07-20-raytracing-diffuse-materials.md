---
title: Raytracing - Diffuse Materials
date: 2018-07-20
tags:
- Computer Graphics
- Raytracing
- Ray Tracing in One Weekend
- PBR
---
<img src="{{ site.url }}/images/raytracing-diffuse-final.jpg" width="640"  style="display:block; margin:auto;">
<br>
<!-- ![]({{ site.url }}/images/raytracing-diffuse-final.jpg) -->
Finish reading [Ray Tracing in One Weekend](http://in1weekend.blogspot.com/2016/01/ray-tracing-in-one-weekend.html) (RTIOW) Chapter 7. Worked through the algorithm of simulating diffuse material with raytracing. Breakdown topics of diffuse reflection, random reflecting ray generation and rejection sampling in unit sphere.
<!-- <div style="text-align:center">
</div> -->
<br>

<!--
https://blog.csdn.net/libing_zeng/article/details/72599041
https://blog.csdn.net/libing_zeng/article/details/54428306
-->
# Diffuse Material and Diffuse Reflection
Object with a diffuse material **doesn't emit light** but take on the colors from the surroundings(background/sky light), and **modulate** _(alter the amplitude or frequency of an electromagnetic wave or other oscillation in accordance with the variations of a second signal, typically one of a lower frequency)_ the colors with its own intrinsic color.

<img src="https://upload.wikimedia.org/wikipedia/commons/e/e1/FullMoon2010.jpg" width="320"  style="display:block; margin:auto;">

> The visibility of objects, excluding light-emitting ones, is primarily caused by **diffuse reflection** of light: it is diffusely-scattered light that forms the image of the object in the observer's eye. Another term commonly used for this type of reflection is "**light scattering**".

The light that reflects off a diffuse surface has its direction randomized.

<img src="https://upload.wikimedia.org/wikipedia/commons/7/79/Diffuse_reflection.PNG
" width="320"  style="display:block; margin:auto;">

> Diffuse reflection is the reflection of light from a surface such that a ray incident on the surface is scattered at many angles rather than at just one angle as in the case of specular reflection.

> An ideal diffuse reflecting surface is said to exhibit [Lambertian reflection](https://en.wikipedia.org/wiki/Lambertian_reflectance), meaning that there is equal luminance when viewed from all directions lying in the half-space adjacent to the surface. More technically, the surface's luminance is **isotropic**, and the luminous intensity obeys [Lambert's cosine law](https://en.wikipedia.org/wiki/Lambert%27s_cosine_law).

In RTIOW we are using a simple algorithm to approximate mathematically ideal **Lambertian reflection**.

<img src="https://upload.wikimedia.org/wikipedia/commons/b/bd/Lambert2.gif
" width="320"  style="display:block; margin:auto;">
>Figure: The rays represent luminous intensity, which varies according to Lambert's cosine law for an ideal diffuse reflector.

Also the light got absorbed a lot rather than reflected, and this is called **attenuation** that caused by light scattering. The darker the surface is the more light-absorption (eg. [Vantablack](https://en.wikipedia.org/wiki/Vantablack) is the darkest artificial substance known, absorbing up to 99.965% of radiation in the visible spectrum).

<!-- _* Note that in RTIOW, we separates geometry from material and treat them as 2 unlinked classes._ -->

# Simulating Diffuse Material With Raytracing
Now with all the rays that are generated from the origin shooting onto the object, how can we calculate the reflected rays? Since the reflected light direction on diffuse surface is randomized, now we need to generate this random direction to build the reflecting ray.

The solution is to add a small random vector onto the normal vector.

## Algorithm Breakdown
### Build Random Reflecting Ray
From the point $P$ where ray hit on the sphere, we form a unit sphere that is tangent to this hitpoint. $\vec{N}$ is the surface normal at $P$.

<img src="{{ site.url }}/images/raytracing-diffuse-fig1.png" width="720"  style="display:block; margin:auto;">

Next we need to form another unit sphere centered at ray/view origin (simple on calculation, also will prevent reflecting direction from pointing into the sphere), and pick a random point $S$ in this unit sphere to build the small random vector $\vec{E}$. With all of them we can calculate the point $R$ (```vec3 target```) which is on the random reflecting direction:

$$R = P+\vec{N}+\vec{E}$$

Then from $P$, we build another ray with the reflecting direction to shoot to next object, and repeat this process again and again, until it has nothing to hit on. At this moment we return the color of the background.

```c
vec3 color(const ray& r, hitable *world){
    hit_record rec;
    if(world->hit(r, 0.001, MAXFLOAT, rec)){
        vec3 target = rec.p + rec.normal + random_in_unit_sphere();
        /* for random_in_unit_sphere() see next section */
        return 0.5 * color(ray(rec.p, target-rec.p), world); // * 0.5
    }
    else{
        // return background color
    }
}
```

### Light Attenuation
Note that every time the ray reflect, we multiply a value 0.5 (light bouncing rate) to the color to simulate the absorption of the light (light attenuation) when it bounces around diffuse materials.

<img src="{{ site.url }}/images/raytracing-diffuse-sample 100-bounce 0.1.jpg" width="640"  style="display:block; margin:auto;">
<div style="text-align:center">
bouncing rate 10%
</div>
<br>
<img src="{{ site.url }}/images/raytracing-diffuse-sample 100-bounce 0.5.jpg" width="640"  style="display:block; margin:auto;">
<div style="text-align:center">
bouncing rate 50%
</div>
<br>
<img src="{{ site.url }}/images/raytracing-diffuse-sample 100-bounce 0.9.jpg" width="640"  style="display:block; margin:auto;">
<div style="text-align:center">
bouncing rate 90%
</div>
<br>

## Rejection Sampling In Unit Sphere
To find this random point $S$ in a unit sphere, we are using the easiest algorithm **rejection method**.

An [example](https://en.wikipedia.org/wiki/Rejection_sampling#Examples) in 2D case:
> As a simple geometric example, suppose it is desired to generate a random point within the unit circle. Generate a candidate point $(x,y)$ where $x$ and $y$ are independent uniformly distributed between −1 and 1. If it happens that $x^{2}+y^{2}\leq 1$ then the point is within the unit circle and should be accepted. If not then this point should be rejected and another candidate should be generated.
<img src="https://upload.wikimedia.org/wikipedia/en/6/66/Circle_sampling.png" width="200"  style="display:block; margin:auto;">

For each x, y, z coordinates, choose a random value of a uniform distribution between [-1, 1]. If the length of the resulting vector is greater than one, reject it and try again.

``` c
vec3 random_in_unit_sphere(){
    vec3 p;
    do{
        p = 2.0 * vec3(drand48(), drand48(), drand48()) - vec3(1,1,1);
    } while (p.squared_length() >= 1.0);
    return p;
}
```
