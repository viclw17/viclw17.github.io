---
title: Raytracing - Dielectric Materials
date: 2018-08-05
tags:
- Computer Graphics
- Raytracing
- Ray Tracing in One Weekend
- PBR
---
<img src="{{ site.url }}/images/raytracing-dielectric-0.jpg" width="640"  style="display:block; margin:auto;">
Chapter 9. Breakdown topics of refraction optic physics (refractive index, Snell's Law, total reflection) and vector maths about calculating refraction ray. TBC.
<!-- <div style="text-align:center">
</div>
<br> -->
<!-- ![](https://ichef.bbci.co.uk/images/ic/976x549_b/p02vk68y.jpg) -->

# Dielectric Transparent Material
Dielectric material can reflect light and at the same time let the light pass through - refract.

<img src="https://upload.wikimedia.org/wikipedia/commons/1/13/F%C3%A9nyt%C3%B6r%C3%A9s.jpg" width="480"  style="display:block; margin:auto;">

# Refraction
## Refractive Index
Refractive index describes how light propagates through that medium. It is defined as

$$ n={\frac {c}{v}}$$

where $c$ is the speed of light in vacuum and $v$ is the speed of light in the medium.

Refractive index $n$ in some common materials:

- Vacuum 1
- Air 1.000293
- Water 1.333
- Ice 1.31
- Window glass 1.52
- Diamond	2.42

The refractive index determines **how much the path of light is bent**, or refracted, when entering a material. This can be described by **Snell's law of refraction**.

## Snell's Law of Refraction
<img src="https://upload.wikimedia.org/wikipedia/commons/3/3f/Refraction_at_interface.svg" width="240"  style="display:block; margin:auto;">

> [Snell's law](https://en.wikipedia.org/wiki/Snell%27s_law) states that the ratio of the sines of the angles of incidence and refraction is equivalent to the ratio of phase velocities in the two media, or equivalent to the reciprocal of the ratio of the indices of refraction:

$${\frac {\sin \theta _{2}}{\sin \theta _{1}}}={\frac {v_{2}}{v_{1}}}={\frac {n_{1}}{n_{2}}}$$

> with each $\theta$  as the angle measured from the normal of the boundary, $v$ as the velocity of light in the respective medium, $n$ as the refractive index (which is unitless) of the respective medium.


So if here we only consider rendering object with dielectric material (with a refractive index ${n_{dielectric}}$, defined as ```ref_idx``` in code) in a **vacuum environment**, since the $n$ of vacuum is 1, we get
when ray shoots into object,

$$\frac {n_{1}}{n_{2}} = \frac {1}{n_{dielectric}}$$

and when ray shoot through object back into vacuum,

$$\frac {n_{1}}{n_{2}} = {n_{dielectric}}$$

Define the dielectric material class:
```c
class dielectric : public material {
public:
    dielectric(float ri) : ref_idx(ri) {}

    virtual bool scatter(const ray& r_in,
                 const hit_record& rec,
                 vec3& attenuation,
                 ray& scattered) const;

    float ref_idx;
};
```

## Total Internal Reflection
When light travels from a medium with a higher refractive index to one with a lower refractive index, $\theta_{2}$ is larger than $\theta_{1}$. When $\theta_{2}$ is reaching 90 degree and $\theta_{1}$ keeps getting bigger, light could be completely reflected by the boundary.

The largest possible angle of incidence which still results in a refracted ray is called the **critical angle**.

<img src="https://upload.wikimedia.org/wikipedia/commons/5/5d/RefractionReflextion.svg" width="560"  style="display:block; margin:auto;">

# Refraction Vector
With all the knowledge above, we can calculate refraction vector. First we can model the vectors relationship within a **unit circle** to simplify the vector calculation.

<img src="{{ site.url }}/images/raytracing-dielectric-1.png" width="560"  style="display:block; margin:auto;">

$$ \vec r = \vec A + \vec B $$

Note that when doing vector calculation, we have to be aware of both direction and the length/magnitude of the vectors.

$$\vec A = sin\theta_{2} \cdot \vec M$$

$$\vec B = cos\theta_{2} \cdot -\vec N$$

$$\vec M = Normalize(\vec C + \vec v) = (\vec C + \vec v)/sin\theta_{1}$$

$$\vec C = cos\theta_{1} \cdot \vec N$$

Expand and rearrange the equation we get:

$$\vec r = \frac {n_{1}}{n_{2}} \cdot (\vec v + cos\theta_{1} \cdot \vec N) - cos\theta_{2} \cdot \vec N$$

>$\vec r = \frac {n_{1}}{n_{2}} \cdot (\vec v - cos\theta_{1} \cdot \vec N) - cos\theta_{2} \cdot \vec N$ when we make $\vec v$ point away from the hit point.

After normalizing incidence ray direction $\vec v$, we can calculate $cos\theta_{1}$ by
$$dot(\vec v, \vec n) = \vert \vec v\vert \vert \vec n\vert cos\theta_{1} = cos\theta_{1}.$$

Since

$$sin\theta_{2} = \frac {n_{1}}{n_{2}} \cdot sin\theta_{1},$$

we get

$$cos^2\theta_{2} = 1 - sin^2\theta_{2} = 1 - \frac {n_{1}^2}{n_{2}^2} \cdot sin^2\theta_{1} = 1 - \frac {n_{1}^2}{n_{2}^2} \cdot (1 - cos^2\theta_{1}) = 1 - \frac {n_{1}^2}{n_{2}^2} \cdot (1 - dot(\vec v, \vec n))$$

<!-- If $\theta_{2}$ is larger than 90 degree, we will encounter **total reflection** and have no refraction ray - ray will be reflected back into the object.  -->

And the equation becomes:

$$\vec r = \frac {n_{1}}{n_{2}} \cdot (\vec v - dot(\vec v, \vec n) \cdot \vec N) - \sqrt{1 - \frac {n_{1}^2}{n_{2}^2} \cdot (1 - dot(\vec v, \vec n))} \cdot \vec N$$

So $cos^2\theta_{2} = 1 - \frac {n_{1}^2}{n_{2}^2} \cdot (1 - dot(\vec v, \vec n))$ is the **discriminat** of the equation:
- when $discriminat > 0$, we have refracted ray $\vec r$;
- when $discriminat < 0$, we will encounter **total reflection** and have no refraction ray - ray will be reflected back into the object.
- _note that $discriminat = 0$ is the boundary of total reflection when refracted ray is perpendicular to the surface normal - no reflection nor refraction._

Now define $\frac {n_{1}}{n_{2}}$ as ```ni_over_nt```, ```v``` is incidence ray direction, ```n``` is surface normal, ```refracted``` is the refracted ray direction.

```c
bool refract(const vec3& v, const vec3& n, float ni_over_nt, vec3& refracted) {
    vec3 uv = unit_vector(v);
    float dt = dot(uv, n);
    float discriminat = 1.0 - ni_over_nt * ni_over_nt * (1-dt*dt);
    if(discriminat > 0){
        refracted = ni_over_nt * (uv-n*dt) - n*sqrt(discriminat);
        return true;
    }
    else
        return false; // no refracted ray
}
```

TBC.

<!--
```c
class dielectric : public material {
public:
    // 注意，ref_idx是指光密介质的折射指数和光疏介质的折射指数的比值
    dielectric(float ri) : ref_idx(ri) {}

    bool scatter(const ray& r_in,
                 const hit_record& rec,
                 vec3& attenuation,
                 ray& scattered
                 ) const {

        vec3 outward_normal;
        vec3 reflected = reflect(r_in.direction(), rec.normal);

        attenuation = vec3(1.0,1.0,1.0);

        vec3 refracted;
        // ni_over_nt为入射介质的折射指数和折射介质的折射指数的比值
        float ni_over_nt;
        float reflect_prob;
        float cosine;
        // 光线是从球体内部射入空气
        // 所以，入射时的法向量和球的法向量方向相反；
        // 此时入射介质是光密介质，折射介质是光疏介质，所以ni_over_nt = ref_idx
        if (dot(r_in.direction(), rec.normal) > 0){
            outward_normal = -rec.normal;
            ni_over_nt = ref_idx;
            cosine = ref_idx * dot(r_in.direction(), rec.normal) /r_in.direction().length();
        }
        // 光线是从空气射入球体气
        // 所以，入射时的法向量和球的法向量方向同向；
        // 注意，ref_idx是指光密介质的折射指数和光疏介质的折射指数的比值，
        // 此时入射介质是光疏介质，折射介质是光密介质，所以ni_over_nt = 1.0/ref_idx
        else{
            outward_normal = rec.normal;
            ni_over_nt = 1.0 / ref_idx;
            cosine = -dot(r_in.direction(), rec.normal) / r_in.direction().length();
        }

        if(refract(r_in.direction(), outward_normal, ni_over_nt, refracted)){
            // scattered = ray(rec.p, refracted); // refracted
            reflect_prob = schlick(cosine, ref_idx);
        }
        // 出现全反射
        else{
            scattered = ray(rec.p, reflected); // reflected
            // return false;
            reflect_prob = 1.0;
        }

        if(drand48() < reflect_prob) {
            scattered = ray(rec.p, reflected);
        }
        else {
            scattered = ray(rec.p, refracted);
        }
        return true;
    }

    float ref_idx;
};
``` -->
