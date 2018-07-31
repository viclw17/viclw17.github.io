---
title: Raytracing - Reflecting Materials
date: 2018-07-30
tags:
- Computer Graphics
- Raytracing
- Ray Tracing in One Weekend
- PBR
---
<img src="{{ site.url }}/images/raytracing-reflecting3.jpg" width="640"  style="display:block; margin:auto;">
<!-- ![](https://ichef.bbci.co.uk/images/ic/976x549_b/p02vk68y.jpg) -->

# Material Base Class
Objects with different materials scatter the lights in different ways. So material tells how rays interact with the surface.

When abstracting materials as a class, it should include functionalities like:
- taking the received ray ```ray& r_in``` and output reflected ray ```ray& scattered```;
- calculating how much the ray should be attenuated - ```vec3& attenuation```;
- gathering the hit point info and save into a ```hit_record``` struct ```rec```.

And for ```struct hit_record```, it records
- parameter ```t``` of the intersected ray;
- position of intersection point ```p```;
- surface normal of intersection point ```normal```;
- and includes a reference to the material of the hit surface.

So we create an abstract class ```material``` with a definition of a pure virtual function ```scatter()``` that take care of all the material specific functionalities mentioned above. Note that an abstract class constructs no objects but works as a template for its children.
```c
#include "hitable.h"

/*
struct hit_record{
    float t;
    vec3 p;
    vec3 normal;
    material *mat_ptr;
};
*/

class material {
public:
    virtual bool scatter(
        const ray& r_in,
        const hit_record& rec,
        vec3& attenuation,
        ray& scattered) const = 0;
};
```

Here we create ```lambertian``` (diffuse material) and ```metal``` (reflecting material) material classes that inherited from ```material``` class. **Note that the children have to implement the pure virtual function of their base class.**


# Lambertian Material
Lambertian Material is an alternative technical name of **diffuse material**. Here we refractor the code about diffuse material from previous post.
> Lambertian reflectance is the property that defines an **ideal** "matte" or diffusely reflecting surface. The apparent brightness of a Lambertian surface to an observer is the same regardless of the observer's angle of view. More technically, the surface's luminance is **isotropic**, and the luminous intensity obeys **Lambert's cosine law**.

```c
#include "material.h"

vec3 random_in_unit_sphere() {
    vec3 p;
    do{
        float random = drand48();
        p = 2.0 * vec3(random, random, random) - vec3(1,1,1);
    } while (p.squared_length() >= 1.0);
    return p;
}

class lambertian : public material {
public:
    lambertian(const vec3& a) : albedo(a) {}

    virtual bool scatter(
        const ray& r_in,
        const hit_record& rec,
        vec3& attenuation,
        ray& scattered) const {

        vec3 target = rec.p + rec.normal + random_in_unit_sphere();
        scattered = ray(rec.p, target - rec.p);
        attenuation = albedo;
        return true;
    }

    vec3 albedo;
};
```
# Reflecting Materials
- **Polished** - A polished reflection is an undisturbed reflection, like a mirror or chrome.
- **Blurry** - A blurry reflection means that tiny random bumps on the surface of the material cause the reflection to be blurry.
- **Metallic** - A reflection is metallic if the highlights and reflections retain the color of the reflective object.
- **Glossy** - This term can be misused. Sometimes, it is a setting which is the opposite of blurry (e.g. when "glossiness" has a low value, the reflection is blurry). However, some people use the term "glossy reflection" as a synonym for "blurred reflection". Glossy used in this context means that the reflection is actually blurred.

# Polished Reflecting Material
<br>
<img src="https://upload.wikimedia.org/wikipedia/commons/1/10/Reflection_angles.svg" width="200"  style="display:block; margin:auto;">
<br>

```c
#include "material.h"

vec3 reflect(const vec3& v, const vec3& n) {
    return v - 2 * dot(v,n) * n;
}

class metal : public material {
public:
    metal(const vec3& a, float f) : albedo(a) {}

    virtual bool scatter(
        const ray& r_in,
        const hit_record& rec,
        vec3& attenuation,
        ray& scattered) const {

        vec3 reflected = reflect(unit_vector(r_in.direction()), rec.normal);
        scattered = ray(rec.p, reflected);
        attenuation = albedo;
        return (dot(scattered.direction(), rec.normal) > 0);
    }

    vec3 albedo;
};
```

<img src="{{ site.url }}/images/raytracing-reflecting1.jpg" width="640"  style="display:block; margin:auto;">

# Blurry Reflecting Material
```c
#include "lambertian.h" // random_in_unit_sphere()

// new constructor
metal(const vec3& a, float f) : albedo(a) {
    if (f<1) fuzz = f;
    else fuzz = 1;
}

    virtual bool scatter(
        const ray& r_in,
        const hit_record& rec,
        vec3& attenuation,
        ray& scattered) const {

        vec3 reflected = reflect(unit_vector(r_in.direction()), rec.normal);
        scattered = ray(rec.p, reflected + fuzz * random_in_unit_sphere()); // random
        attenuation = albedo;
        return (dot(scattered.direction(), rec.normal) > 0);
    }

    vec3 albedo;
    float fuzz; // random intensity
};
```
<img src="{{ site.url }}/images/raytracing-reflecting2.jpg" width="640"  style="display:block; margin:auto;">
