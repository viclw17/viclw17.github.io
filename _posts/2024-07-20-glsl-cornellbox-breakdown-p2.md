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

# Vertex shader `rect.vert`
`rect.vert` is the only vertex shader needed in the application to process the quad mesh to be displayed on the screen.

```c
#version 330 core
layout (location = 0) in vec3 vPos;
layout (location = 1) in vec2 vtexCoord;

out vec2 texCoord;

void main() {
  texCoord = vtexCoord;
  gl_Position = vec4(vPos, 1.0);
}
```

It defined 2 **input vertex attributes** to receive data from CPU on vertex positions and vertex texture coordinates (UV). Vertex positions are sent into `gl_position` in shader and UV is passed down the pipeline to fragment shader stage by using `out` keyword.

The data preparation (initialization) is done in `rectangle.h` in constructor `Rectangle()`. That includes:

- create and bind **vertex array object** `VAO`, which stores instruction of how to use vertex data
- create and bind **vertex buffer object** `VBO`, which stores the real vertices data that being sent to GPU's memory -- copy vertices array into the buffer (VBO) for OpenGL to use (`glBufferData`)
- create and bind **element buffer object** `EBO`, which stores vertex indices
- lastly set **vertex attributes pointers** (`glVertexAttribPointer`)

and the drawing is done in `Rectangle::draw()` by:

- activate shader program
- bind VAO
- draw elements
- unbind VAO
- deactivate shader program

> More details to refresh all the basic concepts at [Hello Triangle - learnopengl.com](https://learnopengl.com/Getting-started/Hello-Triangle)



# Fragment shader `pt.frag`

## Overview and main() function

This is the main shader contains function `computeRadiance(in Ray ray_in)` that calculate the path tracing result.

It includes many other shader files:

- `global.frag` that contains definition of math constants and useful structs - `PI`; `Ray`, `IntersectInfo`, `Material`, `Primitive`, `Light`, etc.
- `uniform.frag` that contains **uniform variables and uniform blocks** definition - `accumTexture`, `stateTexture`, `GlobalBlock`, `CameraBlock`, `SceneBlock`, etc.
  - note the uniform data types used:
    - sampler2D for regular texture
    - usampler2D for random state texture
    - layout(std140) for uniform blocks

```c
uniform sampler2D accumTexture;
uniform usampler2D stateTexture;

layout(std140) uniform GlobalBlock {
  uvec2 resolution;
  float resolutionYInv;
};

layout(std140) uniform CameraBlock {
  vec3 camPos;
  vec3 camForward;
  vec3 camRight;
  vec3 camUp;
  float a;
} camera;

const int MAX_N_MATERIALS = 100;
const int MAX_N_PRIMITIVES = 100;
const int MAX_N_LIGHTS = 100;

layout(std140) uniform SceneBlock {
  int n_materials;
  int n_primitives;
  int n_lights;
  Material materials[MAX_N_MATERIALS];
  Primitive primitives[MAX_N_PRIMITIVES];
  Light lights[MAX_N_LIGHTS];
};
```

- `rng.frag` for random number generators and related functions
  - `uint xorshift32(inout XORShift32_state state)`
  - `float random()`
    - return random number between 0, 1
  - `void setSeed(in vec2 uv)`
    - sample the random state texture and use the value as the random number generator seed for each pixel/texel
- `raygen.frag` to generate camera ray
  - `Ray rayGen()`
- `util.frag ` for matrix transformations
  - `float atan2()`
  - `vec3 worldToLocal()`
  - `vec3 localToWorld()`
- `intersect.frag` to define functions to check geometry intersection
  - `bool intersectSphere()`
  - `bool intersectPlane()`
- `closest_hit.frag` added one layer of encapsulation for checking intersection for all primitives in the scene
  - `bool intersect_each()`
  - `bool intersect()`
- `sampling.frag` to define functions to sample points on various surfaces
  - `vec3 sampleHemisphere()`
  - `vec3 sampleCosineHemisphere()`
  - `vec3 samplePlane()`
  - `vec3 sampleSphere()`
  - `vec3 samplePointOnPrimitive()`
- `brdf.frag` to define materials
  - `float fresnel()`
  - `vec3 BRDF()`
  - `vec3 sampleBRDF()`

These functions will be broken down later when necessary.

Shader started at `main()` function. 

```c
void main() {
    // set RNG seed
    setSeed(texCoord);

    // generate initial ray
    vec2 uv = (2.0*(gl_FragCoord.xy + vec2(random(), random())) - resolution) * resolutionYInv;
    uv.y = -uv.y;
    float pdf;
    Ray ray = rayGen(uv, pdf);
    float cos_term = dot(camera.camForward, ray.direction);

    // accumulate sampled color on accumTexture
    vec3 radiance = computeRadiance(ray) / pdf;
    color = texture(accumTexture, texCoord).xyz + radiance * cos_term;

    // save RNG state on stateTexture
    state = RNG_STATE.a;
}
```

It receives UV `texCoord` passed down by vertex shader; it also defines 2 uniform attributes for output:

```c
in vec2 texCoord;

layout (location = 0) out vec3 color;
layout (location = 1) out uint state;
```

On CPU side, the 2 uniform are set by

```c
// set uniforms
pt_shader.setUniformTexture("accumTexture", accumTexture, 0);
pt_shader.setUniformTexture("stateTexture", stateTexture, 1);
```

Inside the main function, it first calls `setSeed(texCoord)` feeding in screen quad texture coordinates.

```c
void setSeed(in vec2 uv) {
    RNG_STATE.a = texture(stateTexture, uv).x;
}
```

`XORShift32_state RNG_STATE` is defined in `global.frag`. It is set by `setSeed()` by sampling a `stateTexture`. `stateTexture` is a texture uniform passed in from CPU. It was initially setup in `Renderer()` in `renderer.h` with `uniform_int_distribution`.

In the end the random value is saved back to stateTexture (of each texel)

UV on the camera plane is calculated relative to the defined resolution, and **flipped on y axis**.

Camera ray is generated by `rayGen(uv, pdf)`. *pdf* is filled with probability weight of the generated ray. (*need more digging here, tbd)

Computed radiance value then divides *pdf* from ray generation, and added into the `accumTexture` value, after scaled by cosine term (*tbd).

## Compute radiance function
This code is using the similar approach from [Notes from A Path Tracing Workshop](https://viclw17.github.io/2024/07/15/path-tracing-workshop-note) since they both implement path tracing in shader code that do not support recursion.

First, it initializes couple of variables to be tracked and updated during the process:
- `float russian_roulette_prob = 1`
- `vec3 color = vec3(0)`
- `vec3 throughput = vec3(1)`

Now entering main for loop. It will loop till `MAX_DEPTH`.

### Russian Roulette
This is one of the new techniques incorporated in this implementation.

At the beginning of each loop, generate a random float between 0 and 1, if it is larger than `russian_roulette_prob`, break the loop aka terminate the path. If the loop is not broken, divide the throughput by `russian_roulette_prob`(which equals to scaling up the throughput).

```c
if(random() >= russian_roulette_prob) {
    break; // this path ends here
}

throughput /= russian_roulette_prob;
```

At the end of each loop, if ray intersects the scene, after the throughput is updated, we get the max element of throughput (because it is a RGB value), and set that to be the newly updated `russian_roulette_prob`.

```c
float throughput_max_element = max(max(throughput.x, throughput.y), throughput.z);

russian_roulette_prob = min(throughput_max_element, 1.0);
```

The theory behind it is better to understand heuristically. As the light keeps bouncing and the path keeps growing, the **rarity of later bounces** is growing continuously. 

Here we are choosing `russian_roulette_prob` dynamically by computing it at each bounce according to possible color contribution, aka the throughput. 

The more light bounces, the smaller the throughput, the smaller russian_roulette_prob will be set (min(throughput_max_element, 1.0)), and more possible that the random value drawn is larger than it then break the loop. When it didn't break the loop, the division by `russian_roulette_prob` compensates the throughput for times where we miss the path.

> Russian Roulette randomly terminates a path with a probability inversely equal to the throughput. So paths with low throughput that won't contribute much to the scene are more likely to be terminated.
> 
> If we stop there, we're still biased. We 'lose' the energy of the path we randomly terminate. To make it unbiased, we boost the energy of the non-terminated paths by their probability to be terminated. This, along with being random, makes Russian Roulette unbiased.
> 
> ...In the end, Russian Roulette is a very simple algorithm that uses a very small amount of extra computational resources. In exchange, it can save a large amount of computational resources. [source](https://computergraphics.stackexchange.com/questions/2316/is-russian-roulette-really-the-answer)


### Evaluate radiance
When ray intersects the scene, get the `hitPrimitive` and `hitMaterial` from `IntersectInfo` struct. 

If the `hitMaterial` emissive is larger than 0, meaning it's object **is a light source** and we will count its color into the final radiance contribution and terminate the path.

```c
if(any(greaterThan(hitMaterial.le, vec3(0)))) {
    color += throughput * hitMaterial.le;
    break;
}
```

If hit object is not a light source, we **sample the BRDF of the hit surface**. 

Note that `wo` is the vector of bounced light leaving/heading *outward* of the surface, and `wi` is the vector of bounced light arriving/heading *inward* the surface. Hence **the o and i subscripts**. Since ray is shooting from the camera toward the scene that means `wo` is **the opposite direction of ray**:

```c
vec3 wo = -ray.direction;
vec3 wo_local = worldToLocal(wo, info.dpdu, info.hitNormal, info.dpdv);
```

BRDF sampling:

```c
 // BRDF Sampling
float pdf;
vec3 wi_local;

// material contains type ID, kd, le emissive
vec3 brdf = sampleBRDF(wo_local, wi_local, hitMaterial, pdf);

// prevent NaN
if(pdf == 0.0) {
    break;
}

vec3 wi = localToWorld(wi_local, info.dpdu, info.hitNormal, info.dpdv);

// update throughput
float cos_term = abs(wi_local.y);
throughput *= brdf * (1 / pdf) * cos_term;
```

Here we can see throughput is scaling down **by BRDF, by sampling pdf, and by surface cosine**.

Note that the brdf sampling `pdf` is located inside the brdf shader file, as well as the bounced-off light direction `wi_local`, which is very organized. This direction, together with the hit point position, are later to be used to generate next bounced ray.

In the end, if not intersecting anything, simply:

```c
if(intersect(ray, info)) { //...
} else {
    color += throughput * vec3(0);
    break;
}

return color;
```

# Output shader
`accumTexture` is sampled at this stage and `samplesInv` uniform is passed in to normalize the result by dividing the sample amount. 

```c
#version 330 core

uniform float samplesInv;
uniform sampler2D accumTexture;

in vec2 texCoord;
out vec4 fragColor;

void main() {
  vec3 color = texture(accumTexture, texCoord).xyz * samplesInv;
  // gamma
  fragColor = vec4(pow(color, vec3(0.4545)), 1.0);
}
```


```c
void render() {
    // ...
    switch (mode) {
        case RenderMode::Render:
        glBindFramebuffer(GL_FRAMEBUFFER, accumFBO);
        switch (integrator) {
            case Integrator::PT:
            rectangle.draw(pt_shader);
            break;
        }
        glBindFramebuffer(GL_FRAMEBUFFER, 0);

        // update samples
        samples++;

        // output
        output_shader.setUniform("samplesInv", 1.0f / samples);
        rectangle.draw(output_shader);
        break;

// ...
}
```

# Ray generation `raygen.frag`
Here breakdown some details in camera ray generation function. Note that all `camPos`, `camForward` etc. vectors are passed in by `CameraBlock` uniform (located in uniform.frag). 

```c
Ray rayGen(in vec2 uv, out float pdf) {
    vec3 pinholePos = camPos + 1.7071067811865477 * camForward; //?
    vec3 sensorPos = camPos + uv.x * camRight + uv.y * camUp;

    Ray ray;
    ray.origin = camPos;
    ray.direction = normalize(pinholePos - sensorPos);
    float cosineTheta = dot(ray.direction, camForward);
    pdf = 1.0 / pow(cosineTheta, 3.0);
    return ray;
}
```

Note that the ray direction is picked by pointing from pixel position on the screen `sensorPos` to `pinholePos`. Here it seems the pinhole position is moved forward by a small amount from the camera position. Sensor position is at `z = camPos`, so this way `normalize(pinholePos - sensorPos)` will guarantee **the ray is shooting into the scene**.

Although the radiance is arriving at the pinhole which is a point, the energy is actually distributed across the sensor plane to form the image. Radiance is evaluated with differential solid angle (direction) not differential area. As the image is formed on a plane rather than a sphere surface, radiance coming from each ray is contributing different amount to each pixel on the image plane. 

There is a ratio between differential solid angle and differential area which is mentioned in PBRT at [4.2.3 Integrals over Area](https://pbr-book.org/4ed/Radiometry,_Spectra,_and_Color/Working_with_Radiometric_Integrals#IntegralsoverArea):


$$dw = \frac{ dA cos \theta} {r^2}$$

$$dA = \frac{ dw r^2 } {cos \theta} = \frac{ dw (cos \theta)^2 } {cos \theta} = \frac{ dw } {cos^3 \theta}$$

In other words, radiance arriving on a point (dA) aka a pixel is in proportion to the radiance arriving from a direction (dw).

I found a similar scenario like [12.2.2 Texture Projection Lights](https://www.pbr-book.org/4ed/Light_Sources/Point_Lights#TextureProjectionLights) from PBRT, which also contains a great explanation for the derication of *pdf* in the code above.

> ... differential area $dA$ is converted to differential solid angle $dw$ by multiplying by a $cos \theta$ factor and dividing by the squared distance. 
> 
> Because the plane we are integrating over is at $z = 1$, the distance from the origin to a point on the plane is equal to $1/cos\theta$ and thus the aggregate factor is $cos^3\theta$;

Therefore, the generated camera ray has its special pdf relative to solid angle (direction). We can call it pdfDir to be the same as PBRT:


```c
// PBRT - src\cameras\perspective.cpp
void PerspectiveCamera::Pdf_We(const Ray &ray, Float *pdfPos,
                               Float *pdfDir) const {

    // ...

    // Compute lens area of perspective camera
    Float lensArea = lensRadius != 0 ? (Pi * lensRadius * lensRadius) : 1;
    *pdfPos = 1 / lensArea;
    *pdfDir = 1 / (A * cosTheta * cosTheta * cosTheta);
}
```

Each camera ray has a particular probability density ralative to direction, which is what radiance is evaluated; so the end result radiance have to divide this pdf:

```c
void main() {
    // ...

    float pdf;
    Ray ray = rayGen(uv, pdf);
    float cos_term = dot(camForward, ray.direction);

    // accumulate sampled color on accumTexture
    vec3 radiance = computeRadiance(ray) / pdf;
    color = texture(accumTexture, texCoord).xyz + radiance * cos_term;

    // ...
}
```

This part took me a while to chase down an explanation. It is still making my head spin but I will leave those notes here for future revisit.


<!-- > Although most cameras are substantially more complex than the pinhole camera, it is a convenient starting point for simulation. The most important function of the camera is to define the portion of the scene that will be recorded onto the film. In Figure 1.2, we can see how connecting the pinhole to the edges of the film creates a double pyramid that extends into the scene. Objects that are not inside this pyramid cannot be imaged onto the film. Because actual cameras image a more complex shape than a pyramid, we will refer to the region of space that can potentially be imaged onto the film as the **viewing volume**.

> Another way to think about the pinhole camera is to **place the film plane in front of the pinhole** but at the same distance (Figure 1.3). Note that connecting the hole to the film defines exactly the same viewing volume as before. Of course, this is not a practical way to build a real camera, but **for simulation purposes it is a convenient abstraction**. When the film (or image) plane is in front of the pinhole, the pinhole is frequently referred to as the eye.

> Therefore, an important task of the camera simulator is to take a point on the image and generate rays along which incident light will contribute to that image location. Because a ray consists of an origin point and a direction vector, this task is particularly simple for the pinhole camera model of Figure 1.3: it uses the pinhole for the origin and the vector from the pinhole to the imaging plane as the ray’s direction. -- From PBRT[text](https://pbr-book.org/4ed/Introduction/Photorealistic_Rendering_and_the_Ray-Tracing_Algorithm#CamerasandFilm) -->




# BRDF sampling `brdf.frag`
The code separate the BRDF evaluation, and switches it by 3 classic material types - lambert, mirror and glass.

Note that the sampling of the lambert surface is also done here which will provides the corresponding pdf. The pdf will be passed outside into path tracing loop and used for monte carlo estimation by dividing it.

```c
vec3 sampleBRDF(in vec3 wo, out vec3 wi, in Material material, out float pdf) {
    switch(material.brdf_type) {
    // lambert
    case 0:
        // receiving sample pdf
        // then pass back out to caller
        wi = sampleCosineHemisphere(random(), random(), pdf);
        // return albedo, kd kf
        return material.kd * PI_INV;
        break;

    // mirror
    case 1:
        pdf = 1.0;
        wi = reflect(-wo, vec3(0, 1, 0));
        return material.kd / abs(wi.y);
        break;

    // glass
    case 2:
        pdf = 1.0;

        // set appropriate normal and ior
        vec3 n = vec3(0, 1, 0);
        float ior1 = 1.0;
        float ior2 = 1.5;
        if(wo.y < 0.0) {
            n = vec3(0, -1, 0);
            ior1 = 1.5;
            ior2 = 1.0;
        }
        float eta = ior1 / ior2;

        // fresnel
        float fr = fresnel(wo, ior1, ior2);

        // reflection
        if(random() < fr) {
            wi = reflect(-wo, n);
        }
        // refract
        else {
            wi = refract(-wo, n, eta);
            // total reflection
            if(wi == vec3(0)) {
                wi = reflect(-wo, n);
            }
        }

        return material.kd / abs(wi.y);
        break;
    }
}
```

# Final thoughts

TBC