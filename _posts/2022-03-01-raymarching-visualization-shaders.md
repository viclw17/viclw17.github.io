---
title: "Raymarching Visualization Shaders"
layout: post
image: 2022-03-01-raymarching-visualization-shaders/1.png
---

<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/NlSSDy?gui=true&t=10&paused=false&muted=true" allowfullscreen style="display:block; margin:auto;"></iframe>

During my study on implementing raymarching shadertoy in Unreal Engine, I constantly got confused by the algorithm simply because I cannot keep a clear and stable visual model of it in mind. A good way to drill how the algorithm works is to output some debug images to visualze the scene. 

## A refresher
> The shader program runs for every pixel on the canvas IN PARALLEL. This is extremely important to keep in mind.

An amazing shadertoy raymarching tutorial:

*click to interact!*

<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/4dSfRc?gui=true&t=10&paused=false&muted=true" allowfullscreen style="display:block; margin:auto;"></iframe>

<img src="{{ site.url }}/images/2022-03-01-raymarching-visualization-shaders/2.png" style="display:block; margin:auto;">


## Raymarching Function
### Function input output
The function performs the marching action to draw the scene. This means it will require the **ray information** to know where the marching started and what direction it is marching towards - **ray origin** ```ro``` and **ray direction** ```rd```. The function usually returns the **total marching distance** (```float```).

```glsl
float Raymarching(vec3 ro, vec3 rd);
```

However utilizing **GLSL structs** can pass in-and-out much more data as packages:

```glsl
struct Ray { 
    vec3 ori; 
    vec3 dir;  
};

struct Hit { 
    vec3 pos; 
    float dst; 
    int iter; 
};

Hit Raymarching(Ray ray);
```
Now the ray information is passed in as a ```Ray``` struct, and the ```Hit``` struct is passed back to us with:
- the marching end point position, 
- the total marching distance, and 
- the total iteration.

### Function definition

```glsl
#define MAX_ITERATIONS 100
#define MIN_DISTANCE .001
#define FAR_PLANE    100.

Hit Raymarching(Ray ray) {
    float dO  = 0.; // distance to origin
    float dS = 0.; // distance to scene
    int iteration = 0; // iteration counter
    vec3 position;
    
    // mouse.y debug
    float y = iMouse.y / iResolution.y;
    if(iMouse.y == 0.) y = 1.;
    int steps = int(y * float(MAX_ITERATIONS));
    
    for(int i = 0; i < steps; i++) {
        iteration = i;

        // current marching position
        position = ray.ori + ray.dir * dO;
        dS = sdScene(position);

        // marching forward!
        dO += dS; // sphere tracing
        //dO += .1; // stepping

        // temination
        if(dS <= MIN_DISTANCE || dO > FAR_PLANE) {
            break;       
        }   
    }   
    return Hit(position, dO, iteration);
}
```

## Other boilerplate code
### Scene definition
```glsl
float sdSphere(vec3 p, vec3 pos, float r) {
    return length(pos - p) - r;   
}

float sdFloor(vec3 p, float y) {
    return p.y - y;  
}

//Smooth Minimum by iq
//Source: http://www.iquilezles.org/www/articles/smin/smin.htm
float smin( float a, float b, float k ){
    float h = clamp( 0.5+0.5*(b-a)/k, 0.0, 1.0 );
    return mix( b, a, h ) - k*h*(1.0-h);
}

float sdScene(vec3 p) {
    float sdf = 0.;
    float sdFloor = sdFloor(p, 0.1);
    sdf = smin(sdFloor, sdSphere(p, vec3( 1.5 * sin(iTime), 2.0, 5.1), 1.), .5);
    sdf = smin(sdf,     sdSphere(p, vec3(-1.5 * sin(iTime), 0.5, 5.1), 1.), .5);
    return sdf;
}
```

### MainImage (Raymarching() funciton call and final colors)
```glsl
void mainImage( out vec4 fragColor, in vec2 fragCoord ){
	vec2 uv = (fragCoord - iResolution.xy * .5) / iResolution.y;
    vec3 ori = vec3(0.,1.,0.);
    // normalize!
    vec3 dir = normalize(vec3(uv, 1.));
    
    Ray ray = Ray(ori,dir);
    Hit scn = Raymarching(ray);
    
    // mouse.x debug
    float x = iMouse.x / iResolution.x;
    if(iMouse.x == 0.) x = .5;   
    
    vec3 col;
    if((fragCoord.x / iResolution.x) <= x) {     
        // debug depth
        float dist = scn.dst / FAR_PLANE; 
        col = vec3(dist * 5.);

        // debug position
        //col += floor(mod(scn.pos,.5)/.5 + .1) * (1./(scn.pos.z - 1.));
        col += smoothstep(0.9,1.0,mod(scn.pos,.5)/.5) + smoothstep(0.9,1.0,mod(scn.pos,-.5)/-.5);

    } else {
        // debug iteration
        col = vec3(float(scn.iter))/float(MAX_ITERATIONS);     
    }
	fragColor = vec4(col,1.);
}
```

## Sphere tracing and stepping
Technically there are 2 implementation details of raymarching.

We can brute-force the marching by just stepping forward for **a fixed distance**

```glsl
Hit Raymarching(Ray ray) {
    //...
    for(int i = 0; i < steps; i++) {
        //...
        dO += .1; // stepping
        //...
    }
}
```

Then you will have something like this:

<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/ssBBzd?gui=true&t=10&paused=false&muted=true" allowfullscreen style="display:block; margin:auto;"></iframe>

About shader above:
- mouse.x to push the iteration
- mouse.y to scale the stepping distance
- left side rgb grid to visualize world position of sdf traced points + depth (distance from the camera to FAR_PLANE)
- right side white gradient to visualize the iterations (how many times the for loop ran to calculate the pixel)

in this case to have a smooth shading, we have to push for:
- small stepping distance (mouse right)
- more iterations to reach the far plane (mouse up)

which will be very **costly and inefficient**, and if the stepping distance is too large, the shape will be **broken into slices**.

<img src="{{ site.url }}/images/2022-03-01-raymarching-visualization-shaders/3.png" style="display:block; margin:auto;">

However this approach could be useful for the situation where we do not have a defined sdf of the shape we want to render. For example when we are using a volume texture to represent the shape and want to do **volumetric rendering**:

<img src="{{ site.url }}/images/2022-03-01-raymarching-visualization-shaders/vol.gif" style="display:block; margin:auto;">

### Sphere tracing
Since the scene is defined using SDF, we can alter the marching distance with the SDF returned values. 

*click to interact!*

<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/4dKyRz?gui=true&t=10&paused=false&muted=true" allowfullscreen style="display:block; margin:auto;"></iframe>

This means, for example, if there is a shape that is in the direction of the ray that is marching from a particular pixel and is *obviously* the closest, then **only 1 iteration is needed** for this pixel's ray to arrive at the shape surface (just march forward by ```dS```).

```glsl
Hit Raymarching(Ray ray) {
    //...
    for(int i = 0; i < steps; i++) {
        position = ray.ori + ray.dir * dO;
        dS = sdScene(position);
        dO += dS; // sphere tracing
        //...
    }
}
```

<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/fsSBRc?gui=true&t=10&paused=false&muted=true" allowfullscreen style="display:block; margin:auto;"></iframe>

About shader above:
- mouse.y to push the iteration
- left side rgb grid to visualize world position of sdf traced points + depth (distance from the camera to FAR_PLANE)
-  right side white gradient to visualize the iterations (how many times the for loop ran to calculate the pixel)

We can see the closer the end of ray is approaching the surface, the more iterations are needed (whiter). But majority of the pixels are breaking out the for loop way earlier thanks to the SDF, and used little iterations (black).

END