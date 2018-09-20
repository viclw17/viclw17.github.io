---
title: Siggraph 2018 Archive
date: 2018-08-12
tags:
---

# Registration

---

# Sunday
## Vulkan
Vulkan course / webgl three.js course

## It's a material world
Eye ball rendering - Disney
Multi scattering material with neural network - China
Prelit - Animal Logic

## Modern Production Renderer
- Arnold - Solid Angle/ Sony Imagework
- Manuka - Wete Digital
- Hyperion - Disney
- RenderMan - Pixar

Denoiser plugin

---

# Monday
## Advances in reak-time rendering in games
PC occur the most innovation
close-to metal api, dX12/Vulkan..
new pip, raytracing

### far cry 5
deferred
terrain/water/GI/PB-lighting/values
water
g-buffer water-prepass  
depth w/ water or depth w/o water
differred lighting water translucency post-processing
SSLR
SSAO
PB day cycle

### Material ADVANCED
call of duty wwii

### Bokeh, deph of field DOF
Epic, Gillaume, graphic Engineer
problem solving, pixel manipulation, detail-driven for the artifacts
really need to write own renderer with OpenGL, to learn realtime rendering techs
write own raytracer to learn fundamental maths and revise cpp, memory management
great problem-breakdown skill
blurry glass effect? type of bokeh?
really think through career situation and plan longer
for algorithm study, how about at least go thru all the brute force solutions first?
...
artifact -> gather sampling density change
hybrid scattering, indirect scattering pass,
scatter occlusion
hole fill
full-res scene color, depth, velocity
prepare
    setup
    TAA
        downsampling -> reduce
-> convolve fg
-> convolve bg
-> gather hole filling
-> brute force gather in focus
recombine
TAA
sub pixel accurate for intersection model
performance
question about post-processing in vr? psvr?
Bloom & VFX emissive mat?
occlusion? half of frame time on depth test, psvr razorGPU, normal?
GPU profiling general
VR optimization general

### Matt Pharr
Offline/realtime
trace the right ray.
use pr for some, then Denoiser / choose ray wisely
choose random num cleverly
monte carlo AO, hemisphere uniform sampling
monte carlo biased estimator, which preference on direction/area, devide by P(), possibility, cosine weighted hemisphere sampling
this is more efficient and less error, slightly more efficient
7 uniform = 4 cosine samples
Practice sampling with rendering AO! No need for colors.
variance reduce linearly with sample count
variance ratio 1.74
rays needed for equal error 1.75
variance is squared error
choose rays on visible surface! 0.088
reduce variance to  0.03
BVH, reduce more variance, 0.012425
sample warping
uniform sampling -> low discrepancy sampling
2.57X lower variance, faster convergence, 2.5x rays to halve error
stratified sampling 4x8
ray budget
variance-driven sampling
firefly
modern light transport algorithms are not robust
solution: clamping/regularized ray

## Reception
skybox
CDM, larry
Laika
dneg, up and down
sony imagework
- engineer with vfx bg
- switch between film and game

---

# Tuesday
## For Love of Tech Art
https://s2018.siggraph.org/session/?sess=sess301
https://s2018.siggraph.org/presentation/?id=gensub_287&sess=sess301

## [Technical art of sea of thieves](https://s2018.siggraph.org/presentation/?id=gensub_287&sess=sess301)

We present a dive into a selection of visual techniques and tools developed throughout the production of "Sea of Thieves," ranging from ocean surface rendering and volumetric clouds to Kraken tentacle simulations and dynamic systems of GPU-calculated ropes and pulleys.
Water/Cloud/tentacle/Rope/Lightning/Vomit
(see photos)

## [Reinterpreting memorable characters in 'INCREDIBLES 2'](https://s2018.siggraph.org/presentation/?id=gensub_303&sess=sess301)
modeling, iteration on 1st movie, adjust on character bodies; adjust on topology to align with the muscles
rigging/face rigging ...
*character, always hard!!!*

## [Introduction to Direct-X raytracing](https://s2018.siggraph.org/presentation/?id=gensubcur_104&sess=sess268)
This course is an introduction to **Microsoft's DirectX Raytracing** API.
The first half focuses on ray tracing basics and **incremental**, open-source shader tutorials accessible for novices. The second half covers API specifics for developers integrating ray tracing into existing raster-based applications.
- https://news.developer.nvidia.com/dx12-raytracing-tutorials/
- https://github.com/Microsoft/DirectX-Graphics-Samples
- https://github.com/Microsoft/DirectX-Graphics-Samples/tree/master/Samples/Desktop/D3D12Raytracing

## D3D12 basics, TODO: check the sample code, write a renderer
low level api, control memory allocation, obj lifespans, synchronization
- create source in gpu memory(tex, vertices, constant buffers),
- copy data into them,
- record gpu instructions into a cmd list,
- submit cmd list to a queue for exe by HW,
- sync to know when work has finished

parallel execution- creating a resource
cpu
- create buffers
- upload heap
- write date to heap
- copy resource
- exe cmd list

gpu
- exe cmd list
- copy resource
- signal
- call back cpu ...

cpu
- queue - signal(fence)
- fence
- wait
- back from gpu
- destroy heap

binding models
descriptor = pointer to gpu resource
~ table = indexable array of descriptors
~ heap = area of gpu memory with multi ~ list
root signature define binding convention, used by shaders to locate data that need to access

### Challenge
acceleration structure format is opaque
ray go everywhere, all geo/shaders have to be ready
different shaders want different resource binding

- build acceleration structures
- memory management(estimate)
- compaction
- animation: rigid body/ skinned animation
- instance masking

config raytracing pipeline
- shader tables, arrays of pointers to shaders
    - shader identifier = 'pointer' to a shader, 32 byte blob
    - hit group = {intersection shader, any hit shader, closest hit shader}
    - shader record = {shader identifier, local root arg}
    - shader table = {shader record A}, {shader record B}
- shader table indexing
- hit group indexing
- compile and link shaders
    - hlsl -> dex.exe -> dxil library

> its all problem-solving practice, under the mutual agreement of world is strictly descriptable AKA science&math. so simple and so clean

DispatchRay()!
fallback layer

## Book Signing

## SCRIPTABLE RENDER PIPELINE FROM SCRATCH
https://s2018.siggraph.org/presentation/?sess=sess451&id=gensubcur_135#038;id=gensubcur_135
Rendering allows you to control many aspects of a scene, how it looks, what tone is conveyed, and how it is stylized. In this course you will learn the basics of the Unity Scriptable Render Pipeline by creating a renderer from scratch. This renderer will include opaque and transparent rendering as well as simple lighting.

## Job Fair
Skybox

## [Real-time Live](https://s2018.siggraph.org/conference/conference-overview/real-time-live/)
[IKENEMA](https://ikinema.com/blog/ikinema-live-at-siggraph-2018/#.W3MNjMGvr_8.linkedin)

## [Electronic Theater](https://s2018.siggraph.org/conference/conference-overview/computer-animation-festival/electronic-theater/)

---

# Wednesday
## [Epic Sessions](https://www.unrealengine.com/en-US/events/siggraph-2018/learn-from-our-tech-experts)
### Tech behind 'speed of light'
collaboration - Porsche, Nvidia (HW), Epic
area light, RTRT, real time GI, no light map :)

#### Tools and tech
RTX GPU 2 Quadro RTX 6000 in STL
UE4, Microsoft DXR

#### Ref. acquisition
material captures on location
marks on the walls :)
32bit linear workflow
unclamped ibl ?

#### Data prep
min oh, TA in Epic
inside UE4, no extra modeling
CAD-Datasmith-DCC(maya)-UE4, endless iteration
batch process, *python script*
uv
organizing, many parts and quick turn around, OCD, detail-driven
876 cad parts - 165 mesh groups - 9.46m polygons
clean the holes to provide useless ray calculation

#### Look-dev
matching the materials as realistic as possible
custom material expressions
translucent
cinematic in sequencer
camera work with virtual cam, iPhone
photo studio build in BP

#### Tech about ray tracing
*Unreal broad applications - industrial design, production design, engineering, architecture, virtual capture human, VR, cinematics*
raytracer, soft shadow is big add-on
raster vs raytracing
RT is more suitable for complex effects: GI, refraction, soft shadows, dispersion, scattering
RT easier to implement and use
why RT important now? 40ms
how did it happened - hardware + implementation

implementation...
- DXR, DirectX Ray Tracing API, good! easy to prototype, food perf, allow recursiveness
- hybrid tech, use raster for primary visibility (G-Buffer), use RT for complex effects (reflections, soft shadows, area shadow...)
- denoiser, different for each task. state of art, no machine learning yet; critical!(1 spp -> denoiser)
- TAA & noise, challenge, TAA cause white spots noise, but when blur out gonna lose the reflections

### Fortnite - Advancing The Animation Production Pipeline
Looking to improve your understanding of animation production in Unreal Engine? Follow Epic’s Brian Pohl and Ryan Mayeda as they chart the course of new workflow and pipeline improvements to UE4 while dissecting the Fortnite cinematic trailer one year after its initial release.

In this lecture, you'll learn more about what we've improved, changed, and added to UE4 to dramatically streamline your animation pipeline. We'll examine Perforce setup, source control, **python integration**, sequencer improvements, better production management techniques through UE4's Shotgun integration capabilities, and more!

### Mixed Reality Production using Unreal Engine 4.20
LBE location based entertainment ...

## Pathtracing in Production
### Reduce noise
sampling noise denoiser!

---

# Thursday
## Epic Session
Ask udn
Clean up component
Unreal tech artist.

## Meet Autodesk

## Exhibition
Epic Unreal TA
