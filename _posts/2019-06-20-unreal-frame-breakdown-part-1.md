---
title: "Unreal Frame Breakdown Part 1"
layout: post
image: 2019-06-20-unreal-frame-breakdown-part-1/frame.jpg
---

This is my version of [How Unreal Render One Frame](https://interplayoflight.wordpress.com/2017/10/25/how-unreal-renders-a-frame/). I'm trying to follow this post and profile a frame by myself to learn the deferred rendering pipeline of Unreal Engine.

I build this testing scene following the post. Unreal is set to use **deferred shading**. If set to use **forward shading** the profiling results will be extremely different and I will try to go through in another post. The ```umap``` is named as NewWorld in my test and it contains:

1. 1 directional light, from left-up to right-down
2. 1 static light, white
3. 2 stationary lights, both blue
4. 2 movable lights, one green one red
5. rock mesh labeled as movable
6. all the rest of meshes are static
7. all meshes with shadows on
8. 1 fire particle system
9. volumetric lighting on
10. skybox

# Particle PreRender
Particle simulation on the GPU (only of **GPU Sprites particle** type). It seems the pass costs 2 drawcalls because we have 2 GPU emitters in this particle system.

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/particleGPU.jpg" width="800"  style="display:block; margin:auto;">

```c
 EID  | Event                                                                        | Draw # | Duration (Microseconds)
      |   - GPUParticles_PreRender                                                   | 2      | 36.864
      |    \- GPUParticles_SimulateAndClear                                          | 2      | 23.552
      |      \- ParticleSimulationCommands                                           | 2      | 23.552
      |        \- ParticleSimulation                                                 | 2      | 23.552
115   |          \- DrawIndexed(48)                                                  | 2      | 23.552
121   |           - API Calls                                                        | 3      | 0.00
      |     - ParticleSimulation                                                     | 4      | 13.312
138   |      \- DrawIndexed(48)                                                      | 4      | 13.312
144   |       - API Calls                                                            | 5      | 0.00
148   |     - API Calls                                                              | 6      | 0.00
```

This pass outputs following 2 **Render Target(RT)**:

1. RT0 Particle State Position
2. RT1 Particle State Velocity

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/particle.jpg" width="400"  style="display:block; margin:auto;">

# PrePass
PrePass takes all non-translucent(eg. with opaque or masked materials) meshes and outputs a **depth Z pass** (Z-prepass). Its results are required by **DBuffer** hence "Forced by DBuffer". It could also be forced by **Forward Shading**. The pre-pass may also be used by **occlusion culling**.
```c
 EID  | Event                                                                        | Draw # | Duration (Microseconds)
      |   - PrePass DDM_AllOpaque (Forced by DBuffer)                                | 7      | 239.616
      |    \- BeginRenderingPrePass                                                  | 7      | 32.768
154   |      \- ClearDepthStencilView(D=0.000000, S=00)                              | 7      | 32.768
      |     - BeginRenderingPrePass                                                  | 8      | 0.00
158   |      \- API Calls                                                            | 8      | 0.00

      |     - WorldGridMaterial None 2 instances                                     | 9      | 19.456
182   |      \- DrawIndexedInstanced(2304, 2)                                        | 9      | 19.456
      |     - WorldGridMaterial SM_Rock                                              | 10     | 15.36
188   |      \- DrawIndexed(3684)                                                    | 10     | 15.36
      |     - WorldGridMaterial None 2 instances                                     | 11     | 17.408
194   |      \- DrawIndexedInstanced(5346, 2)                                        | 11     | 17.408
      |     - WorldGridMaterial Wall_400x400                                         | 12     | 41.984
200   |      \- DrawIndexed(36)                                                      | 12     | 41.984
      |     - WorldGridMaterial None 2 instances                                     | 13     | 78.848
206   |      \- DrawIndexedInstanced(192, 2)                                         | 13     | 78.848
      |     - WorldGridMaterial None 2 instances                                     | 14     | 33.792
212   |      \- DrawIndexedInstanced(36, 2)                                          | 14     | 33.792
      |     - FinishRenderingPrePass                                                 | 15     | 0.00
218   |      \- API Calls                                                            | 15     | 0.00

//       |   - ResolveSceneDepthTexture                                                 | 16     | 0.00
// 223   |    \- API Calls                                                              | 16     | 0.00    
```

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/prepass.gif" width="400"  style="display:block; margin:auto;">

Note that chairs/sculptures/window walls/floor+ceiling are all in pairs so they are rendered as "2 instances" per pair and only cause 1 drawcall per pair. This is the proof of **instancing** being a great practice of optimization by saving drawcalls.

Also notice an interesting detail - this pass is using WorldGridMaterial (engine default material) for all the meshes.

<!-- Some render passes in the list appear to be empty, like the **ResolveSceneDepth**, which is for
platforms that actually need “resolving” a rendertarget before using it as a texture. Not applicable in our case. (the PC doesn’t). -->

# ComputeLightGrid*
> Responsible for optimizing lighting in forward shading. According to the comment in Unreal’s source code, this pass “culls local lights to a grid in frustum space. Needed for forward shading or translucency using the Surface lighting mode”. In other words: it assigns lights to cells in a grid (shaped like a pyramid along camera view). This operation has a cost of its own but it pays off later, making it faster to determine which lights affect which meshes. [source](https://unrealartoptimization.github.io/book/profiling/passes-lighting/#computelightgrid)

```c
      |   - ComputeLightGrid                                                         | 17     | 174.08
      |    \- CullLights 30x16x32 NumLights 13 NumCaptures 1                         | 17     | 123.872
233   |      \- Dispatch(480, 1, 1)                                                  | 17     | 17.408
239   |       - Dispatch(1, 1, 1)                                                    | 18     | 12.288
245   |       - Dispatch(1, 1, 1)                                                    | 19     | 11.264
256   |       - Dispatch(8, 4, 8)                                                    | 20     | 82.912
261   |       - API Calls                                                            | 21     | 0.00
      |     - Compact                                                                | 22     | 50.208
270   |      \- Dispatch(8, 4, 8)                                                    | 22     | 50.176
276   |       - API Calls                                                            | 23     | 0.032
```

# BeginOcclusionTests
The term occlusion culling refers to a method that tries to reduce the rendering load on the graphics system by eliminating objects from the rendering pipeline if they are occluded by other objects. There are several methods for doing this.

1. Initiate an **occlusion query**.
2. Turn off writing to the frame and depth buffer, and disable any superfluous state. Modern graphics hardware is thus able to rasterize at a much higher speed (NVIDIA 2004).
3. Render a simple but conservative approximation of the complex object—usually **a bounding box**: the GPU counts the number of fragments that would actually have passed the depth test.
4. Terminate the occlusion query.
5. Ask for the result of the query (that is, the number of visible pixels of the approximate geometry).
6. If the number of pixels drawn is greater than some threshold (typically zero), render the complex object.

[Source](https://developer.nvidia.com/gpugems/GPUGems2/gpugems2_chapter06.html)

<!-- https://forums.unrealengine.com/development-discussion/vr-ar-development/107536-beginocclusiontests-is-this-something-that-could-take-advantage-of-instanced-stereo -->

The pass starts with 2 ShadowFrustumQueries followed by 1 GroupedQueries and 1 IndividualQueries.

```c
      |   - BeginOcclusionTests                                                      | 24     | 249.856
      |    \- ViewOcclusionTests 0                                                   | 24     | 249.856
      |      \- ShadowFrustumQueries                                                 | 24     | 28.672
297   |        \- DrawIndexed(1296)                                                  | 24     | 16.384
301   |         - DrawIndexed(1296)                                                  | 25     | 12.288
303   |         - API Calls                                                          | 26     | 0.00
      |       - ShadowFrustumQueries                                                 | 27     | 34.816
311   |        \- DrawIndexed(36)                                                    | 27     | 13.312
314   |         - DrawIndexed(36)                                                    | 28     | 10.24
317   |         - DrawIndexed(36)                                                    | 29     | 11.264
320   |         - API Calls                                                          | 30     | 0.00

// 321   |       - PlanarReflectionQueries                                              | 31     |

      |       - GroupedQueries                                                       | 31     | 10.24
327   |        \- DrawIndexed(72)                                                    | 31     | 10.24
330   |         - API Calls                                                          | 32     | 0.00

      |       - IndividualQueries                                                    | 33     | 176.128
333   |        \- DrawIndexed(36)                                                    | 33     | 29.696
337   |         - DrawIndexed(36)                                                    | 34     | 12.288
341   |         - DrawIndexed(36)                                                    | 35     | 10.24
345   |         - DrawIndexed(36)                                                    | 36     | 11.296
349   |         - DrawIndexed(36)                                                    | 37     | 7.168
353   |         - DrawIndexed(36)                                                    | 38     | 10.24
357   |         - DrawIndexed(36)                                                    | 39     | 12.288
361   |         - DrawIndexed(36)                                                    | 40     | 7.168
365   |         - DrawIndexed(36)                                                    | 41     | 12.288
369   |         - DrawIndexed(36)                                                    | 42     | 12.288
373   |         - DrawIndexed(36)                                                    | 43     | 9.216
377   |         - DrawIndexed(36)                                                    | 44     | 12.288
381   |         - DrawIndexed(36)                                                    | 45     | 10.24
385   |         - DrawIndexed(36)                                                    | 46     | 13.28
389   |         - DrawIndexed(36)                                                    | 47     | 6.144
393   |         - API Calls                                                          | 48     | 0.00
```

## ShadowFrustumQueries
The first group of 2 ShadowFrustumQueries is for 2 movable point lights, so the frustum is a sphere. The second group of 3 ShadowFrustumQueries is for 2 stationary point lights and 1 directional light (that 1 static light is fully baked), and for these cases the frustum is a truncated pyramid (note that the frustum for directional light is extremely long).

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/ShadowFrustumQueries.gif" width="400"  style="display:block; margin:auto;">

## GroupedQueries
This is for occluded objects. In my scene there is a table mesh behind the wall got fully occluded so it shows up in this step.

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/GroupedQueries.jpg" width="400"  style="display:block; margin:auto;">

## IndividualQueries
This is for all the other objects, and notice occlusion testing is done on bounding boxes.

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/IndividualQueries.gif" width="400"  style="display:block; margin:auto;">

# BuildHZB
This step generate the Hierarchical Z-Buffer. This takes the depth buffer produced during the Z-prepass as in input and creates a mip chain (i.e. downsamples it successively) of depths.

```c
      |   - BuildHZB(ViewId=0)                                                       | 49     | 207.904
      |    \- HZB(mip=0) 1024x512                                                    | 49     | 100.384
      |     - HZB(mip=1) 512x256                                                     | 50     | 41.984
      |     - HZB(mip=2) 256x128                                                     | 51     | 10.24
      |     - HZB(mip=3) 128x64                                                      | 52     | 8.192
      |     - HZB(mip=4) 64x32                                                       | 53     | 8.192
      |     - HZB(mip=5) 32x16                                                       | 54     | 7.168
      |     - HZB(mip=6) 16x8                                                        | 55     | 8.192
      |     - HZB(mip=7) 8x4                                                         | 56     | 7.168
      |     - HZB(mip=8) 4x2                                                         | 57     | 8.192
      |     - HZB(mip=9) 2x1                                                         | 58     | 8.192
```

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/BuildHZB.gif" width="400"  style="display:block; margin:auto;">


# ShadowDepths
This pass is **only for movable objects** as their shadowing situation should be calculated in realtime. All the static object shadows are supposed to be baked in advance to save performance. In this case all the following calculations are done for the rock mesh because it's labelled as movable object.
```c
      |   - ShadowDepths                                                             | 60     | 997.376
      |    \- Atlas0 2048x2048                                                       | 60     | 112.64
      |      \- SetShadowRTsAndClear                                                 | 60     | 23.552
549   |        \- ClearDepthStencilView(D=1.000000)                                  | 60     | 23.552

      |       - NewWorld.DirectionalLight_1                                          | 61     | 31.744
      |        \- PerObject SM_Rock_19 128x84                                        | 61     | 31.744
561   |          \- Dispatch(14, 1, 1)                                               | 61     | 15.36
      |           - WorldGridMaterial SM_Rock                                        | 62     | 16.384
583   |            \- DrawIndexed(3684)                                              | 62     | 16.384

      |       - NewWorld.PointLight_stationary1                                      | 63     | 29.696
      |        \- PerObject SM_Rock_19 128x80                                        | 63     | 29.696
595   |          \- Dispatch(14, 1, 1)                                               | 63     | 13.312
      |           - WorldGridMaterial SM_Rock                                        | 64     | 16.384
613   |            \- DrawIndexed(3684)                                              | 64     | 16.384

      |       - NewWorld.PointLight_stationary2                                      | 65     | 27.648
      |        \- PerObject SM_Rock_19 128x80                                        | 65     | 27.648
625   |          \- Dispatch(14, 1, 1)                                               | 65     | 13.312
      |           - WorldGridMaterial SM_Rock                                        | 66     | 14.336
640   |            \- DrawIndexed(3684)                                              | 66     | 14.336

      |     - Cubemap NewWorld.PointLight_movable2 512^2                             | 67     | 438.272
      |      \- WholeScene MovablePrimitives 512x512                                 | 67     | 438.272
656   |        \- Dispatch(14, 1, 1)                                                 | 67     | 13.312
      |         - CopyCachedShadowMap                                                | 68     | 378.88
675   |          \- DrawIndexed(6)                                                   | 68     | 378.88
      |         - WorldGridMaterial SM_Rock                                          | 69     | 46.08
692   |          \- DrawIndexed(3684)                                                | 69     | 46.08

      |     - Cubemap NewWorld.PointLight_movable1 512^2                             | 70     | 446.464
      |      \- WholeScene MovablePrimitives 512x512                                 | 70     | 446.464
706   |        \- Dispatch(14, 1, 1)                                                 | 70     | 16.384
      |         - CopyCachedShadowMap                                                | 71     | 380.928
725   |          \- DrawIndexed(6)                                                   | 71     | 380.928
      |         - WorldGridMaterial SM_Rock                                          | 72     | 49.152
742   |          \- DrawIndexed(3684)                                                | 72     | 49.152
745   |           - API Calls                                                        | 73     | 0.00
748   |     - PreshadowCache                                                         | 74     |
```
For 1 directional and 2 stationary lights, the shadow depths are written into Atlas0. One light source corresponds to one shadow depth map.

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/ShadowDepths DirectionalStationaryLight.gif" width="400"  style="display:block; margin:auto;">

2 movable lights are treated in different way. They are using cubemaps to record shadow depths. For each light, firstly CopyCachedShadowMap outputs a cubemap without movable objects.

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/ShadowDepths Cubemap NewWorld.PointLight_movable0.gif" width="300"  style="display:block; margin:auto;">

Then Unreal adds the movable objects shadow depths into the cubemaps.

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/ShadowDepths Cubemap NewWorld.PointLight_movable.gif" width="300"  style="display:block; margin:auto;">

# Volumetric Fog*
[Documentation](https://docs.unrealengine.com/en-US/Engine/Rendering/LightingAndShadows/VolumetricFog/index.html)

```c
      |   - InitializeVolumeAttributes                                               | 74     | 6998.016
762   |    \- Dispatch(60, 32, 32)                                                   | 74     | 6998.016
766   |     - API Calls                                                              | 75     | 0.00
      |   - LightScattering 240x127x128  LF                                          | 76     | 7064.576
793   |    \- Dispatch(60, 32, 32)                                                   | 76     | 7064.576
796   |     - API Calls                                                              | 77     | 0.00
      |   - FinalIntegration                                                         | 78     | 3621.888
804   |    \- Dispatch(30, 16, 1)                                                    | 78     | 3621.856
810   |     - API Calls                                                              | 79     | 0.032
```
<!-- InitializeVolumeAttributes output uav0 = LightScattering and uav1 = VBufferB -->

## Initialize Volume Attributes*
This pass calculates and stores fog parameters (scattering and absorption) into the volume texture and also stores a global emissive value into a second volume texture.

Note that in my test scene I put 1 **AtmosphereFog** and 1 **ExponentialHeightFog**. They are different entities and at this pass it is the ExponentialHeightFog got calculated. AtmosphereFog is more like the skybox (or cubemap used for IBL) and will be treated at Atmosphere pass later.

## Light Scattering*
This pass calculates the light scattering and extinction for each cell combining the shadowed directional light, sky light and local lights, assigned to the Light volume texture during the ComputeLightGrid pass above. It also uses temporal antialiasing on the compute shader output (Light Scattering, Extinction) using a history buffer, which is itself a 3D texture, improve scattered light quality per grid cell.

## Final Integration*
This pass simply raymarches the 3D texture in the Z dimension and accumulates scattered light and transmittance, storing the
result, as it goes, to the corresponding cell grid.

# BasePass
This is the main pass rendering **non-translucent materials**, reading and saving **static lighting** to the G-Buffer. As we can see here we have 6 render targets in GBuffer got cleared before rendering.
<!-- Applying DBuffer decals
Applying fog
Calculating final velocity (from packed 3D velocity)
In forward renderer: dynamic lighting -->
```c
      |   - CompositionBeforeBasePass                                                | 80     | 0.00
812   |    \- DeferredDecals DRS_BeforeBasePass                                      | 80     |
      |   - BeginRenderingGBuffer                                                    | 80     | 54.272
819   |    \- ClearRenderTargetView(0.000000, 0.000000, 0.000000, 1.000000)          | 80     | 10.24
820   |     - ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)          | 81     | 11.264
821   |     - ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)          | 82     | 8.192
822   |     - ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)          | 83     | 8.192
823   |     - ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)          | 84     | 8.192
824   |     - ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)          | 85     | 8.192
826   |     - API Calls                                                              | 86     | 0.00

      |   - BasePass                                                                 | 87     | 2639.84
      |    \- M_Basic_Floor Floor_400x400                                            | 87     | 68.608
864   |      \- DrawIndexed(36)                                                      | 87     | 68.608
      |     - M_Chair None 2 instances                                               | 88     | 102.40
883   |      \- DrawIndexedInstanced(5346, 2)                                        | 88     | 102.40
      |     - M_Brick_Clay_Old Wall_400x400                                          | 89     | 465.92
903   |      \- DrawIndexed(36)                                                      | 89     | 465.92
      |     - M_Statue None 2 instances                                              | 90     | 56.32
919   |      \- DrawIndexedInstanced(2304, 2)                                        | 90     | 56.32
      |     - M_Rock_Marble_Polished Floor_400x400                                   | 91     | 367.616
936   |      \- DrawIndexed(36)                                                      | 91     | 367.616
      |     - M_Rock SM_Rock                                                         | 92     | 98.272
970   |      \- DrawIndexed(3684)                                                    | 92     | 98.272
      |     - M_Wood_Walnut Wall_Window_400x400                                      | 93     | 748.544
998   |      \- DrawIndexed(192)                                                     | 93     | 748.544
      |     - M_Wood_Walnut Wall_Window_400x400                                      | 94     | 732.16
1010  |      \- DrawIndexed(192)                                                     | 94     | 732.16
1015  |     - API Calls                                                              | 95     | 0.00
```
The **actual materials** on the objects are finally used in rendering: M_Basic_Floor, M_Chair, M_Brick_Clay_Old, M_Statue, M_Rock_Marble_Polished, M_Rock, M_Wood_Walnut, M_Wood_Walnut. **At the result this pass is deeply affected by shader complexity.** Also notice this time drawcalls are per-material rather than per-mesh intance like in Z-prepass. So during content optimization, be aware of the **number of material slots** on the meshes.

Examples of different G-buffer render targets: material base color/ normals/ material properties/ baked lightings.
<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/g.jpg" width="800"  style="display:block; margin:auto;">

Note that most of the time different channel of the buffers has different information encoded. We can see the details from the engine code in ```DeferredShadingCommon.ush```:
```c
/** Populates OutGBufferA, B and C */
void EncodeGBuffer(...)
{
    ...
	OutGBufferA.rgb = EncodeNormal( GBuffer.WorldNormal );
	OutGBufferA.a = GBuffer.PerObjectGBufferData;

	OutGBufferB.r = GBuffer.Metallic;
	OutGBufferB.g = GBuffer.Specular;
	OutGBufferB.b = GBuffer.Roughness;
	OutGBufferB.a = EncodeShadingModelIdAndSelectiveOutputMask(GBuffer.ShadingModelID, GBuffer.SelectiveOutputMask);

	OutGBufferC.rgb = EncodeBaseColor( GBuffer.BaseColor );
	OutGBufferC.a = GBuffer.GBufferAO;

	OutGBufferD = GBuffer.CustomData;
	OutGBufferE = GBuffer.PrecomputedShadowFactors;
    ...
}
```

# Velocity
Saving velocity of each vertex (used later by motion blur and temporal anti-aliasing).
```c
      |   - RenderVelocities                                                         | 96     | 9.216
1020  |    \- ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)          | 96     | 9.216
1026  |     - API Calls                                                              | 97     | 0.00
      |   - ResolveSceneDepthTexture                                                 | 98     | 0.00
1032  |    \- API Calls                                                              | 98     | 0.00
```

# AO
```c
      |   - LightCompositionTasks_PreLighting                                        | 99     | 4392.928
1034  |    \- DeferredDecals DRS_AfterBasePass                                       | 99     |
1036  |     - DeferredDecals DRS_BeforeLighting                                      | 99     |
1038  |     - DeferredDecals DRS_Emissive                                            | 99     |

      |     - AmbientOcclusionSetup 958x507                                          | 99     | 489.472
1065  |      \- DrawIndexed(3)                                                       | 99     | 489.472
1068  |       - API Calls                                                            | 100    | 0.00
      |     - AmbientOcclusionPS 958x507 SetupAsInput=1 Upsample=0 ShaderQuality=2   | 101    | 1086.464
1085  |      \- DrawIndexed(3)                                                       | 101    | 1086.464
1088  |       - API Calls                                                            | 102    | 0.00
      |     - AmbientOcclusionPS 1916x1014 SetupAsInput=0 Upsample=1 ShaderQuality=2 | 103    | 1909.728
1116  |      \- DrawIndexed(3)                                                       | 103    | 1909.728
1119  |       - API Calls                                                            | 104    | 0.00
1120  |     - DeferredDecals DRS_AmbientOcclusion                                    | 105    |
      |     - ApplyAOToBasePassSceneColor 1916x1014                                  | 105    | 907.264
1140  |      \- DrawIndexed(3)                                                       | 105    | 907.264
1143  |       - API Calls                                                            | 106    | 0.00

1148  |   - ClearDepthStencilView(S=00)                                              | 107    | 10.24
      |   - ClearTranslucentVolumeLighting                                           | 108    | 279.552
1168  |    \- DrawInstanced(4, 64)                                                   | 108    | 279.552
1171  |     - API Calls                                                              | 109    | 0.00
```
This pass LightCompositionTasks_PreLighting mainly calculates AmbientOcclusion. It firstly outputs a low-res AO then a high-res AO, and finally apply the result to the scene color.
<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/ao1.jpg" width="400"  style="display:block; margin:auto;">

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/ao2.jpg" width="400"  style="display:block; margin:auto;">



# Lighting
Here comes to the big lighting pass. Through this pass Unreal calculates and applys lightings to the scene.

## Non Shadowed Lights
At first the pass processes **NonShadowedLights**. NonShadowedLights include

1. simple lights such as **per particle lighting**, and
2. non shadowed normal scene lights.

A difference between the two is that normal scene lights use a **depth bounds test** when rendering, to avoid lighting pixels outside an approximate light volume.

Here in **StandardDeferredSimpleLights**, each DrawIndexed is for each particle light. As we can see they are processed per particle and are pretty expensive. So it's a good idea to avoid using too much per particle lighting.

The lighting is accumulated to the SceneColourDeferred buffer.

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/StandardDeferredSimpleLights.gif" width="400"  style="display:block; margin:auto;">

```c
      |   - DirectLighting                                                           | 110    | 23271.328
      |    \- NonShadowedLights                                                      | 110    | 17419.264
      |      \- BeginRenderingSceneColor                                             | 110    | 0.00
      |       - StandardDeferredSimpleLights                                         | 111    | 17419.264
1214  |        \- DrawIndexed(1296)                                                  | 111    | 1810.432
1223  |         - DrawIndexed(1296)                                                  | 112    | 2066.432
1232  |         - DrawIndexed(1296)                                                  | 113    | 1979.392
1241  |         - DrawIndexed(1296)                                                  | 114    | 2081.792
1250  |         - DrawIndexed(1296)                                                  | 115    | 1941.504
1259  |         - DrawIndexed(1296)                                                  | 116    | 1825.792
1268  |         - DrawIndexed(1296)                                                  | 117    | 1940.48
1277  |         - DrawIndexed(1296)                                                  | 118    | 1861.632
1286  |         - DrawIndexed(1296)                                                  | 119    | 1911.808
      |       - StandardDeferredLighting                                             | 120    | 0.00
      |        \- BeginRenderingSceneColor                                           | 120    | 0.00
1295  |       - InjectSimpleLightsTranslucentLighting                                | 121    |

      |     - IndirectLighting                                                       | 121    | 0.00
1299  |      \- UpdateLPVs                                                           | 121    |
```

## Shadowed Lights
### Inject Translucent Volume
Unreal’s approach to lighting translucent surfaces comprises of injecting light into 2 volume textures. The two textures store a **spherical harmonics representation** of the light (shadowed+attenuated) that reaches each volume cell (texture TranslucentVolumeX) and an approximate light direction of each light source (texture TranslucentVolumeDirX).

The renderer maintains 2 sets of such textures **one for close to the camera props**, that require higher resolution lighting, and **one for more distant objects** where high resolution lighting is not that important.

Note that from the previous NonShadowedLights pass, it seems simple lights do not appear to write to the translucency lighting volumes at all, hence **InjectSimpleLightsTranslucentLighting** is empty.

### Shadowed Lights
This pass is done on each light - 1 directional 2 stationary and 2 movable because they are all casting shadows. For each light this is done with steps:

1. ShadowProjectionOnOpaque
2. InjectTranslucentVolume
3. StandardDeferredLighting

```c
      |     - ShadowedLights                                                         | 121    | 5852.064
      |      \- NewWorld.DirectionalLight_1                                          | 121    | 933.888
1306  |        \- ClearRenderTargetView(1.000000, 1.000000, 1.000000, 1.000000)      | 121    | 14.336
      |         - ShadowProjectionOnOpaque                                           | 122    | 110.592
      |          \- PreShadow SM_Rock_19                                             | 122    | 51.20
      |            \- Stencil Mask Subjects                                          | 122    | 21.504
      |              \- WorldGridMaterial SM_Rock                                    | 122    | 21.504
1326  |                \- DrawIndexed(3684)                                          | 122    | 21.504
1351  |             - DrawIndexed(36)                                                | 123    | 29.696
      |           - PerObject SM_Rock_19                                             | 124    | 59.392
1364  |            \- DrawIndexed(36)                                                | 124    | 21.504
1380  |             - DrawIndexed(36)                                                | 125    | 37.888
1384  |           - API Calls                                                        | 126    | 0.00
      |         - InjectTranslucentVolume                                            | 127    | 247.808
1407  |          \- DrawInstanced(4, 64)                                             | 127    | 135.168
1418  |           - DrawInstanced(4, 64)                                             | 128    | 112.64
      |         - BeginRenderingSceneColor                                           | 129    | 0.00
1424  |          \- API Calls                                                        | 129    | 0.00
      |         - StandardDeferredLighting                                           | 130    | 561.152
1452  |          \- DrawIndexed(3)                                                   | 130    | 561.152

      |       - NewWorld.PointLight_movable2                                         | 131    | 1443.84
      |        \- ClearQuad                                                          | 131    | 67.584
1475  |          \- Draw(4)                                                          | 131    | 67.584
1477  |           - API Calls                                                        | 132    | 0.00
      |         - ShadowProjectionOnOpaque                                           | 133    | 488.448
1505  |          \- DrawIndexed(1296)                                                | 133    | 441.344
      |           - InjectTranslucentVolume                                          | 134    | 47.104
1530  |            \- DrawInstanced(4, 7)                                            | 134    | 29.696
1541  |             - DrawInstanced(4, 3)                                            | 135    | 17.408
1543  |             - API Calls                                                      | 136    | 0.00
      |         - BeginRenderingSceneColor                                           | 137    | 0.00
1549  |          \- API Calls                                                        | 137    | 0.00
      |         - StandardDeferredLighting                                           | 138    | 887.808
1577  |          \- DrawIndexed(1296)                                                | 138    | 887.808

      |       - NewWorld.PointLight_movable1                                         | 139    | 1415.168
                ...

      |       - NewWorld.PointLight_stationary1                                      | 147    | 956.416
      |        \- ClearQuad                                                          | 147    | 61.44
1725  |          \- Draw(4)                                                          | 147    | 61.44
1727  |           - API Calls                                                        | 148    | 0.00
      |         - ShadowProjectionOnOpaque                                           | 149    | 88.064
      |          \- PreShadow SM_Rock_19                                             | 149    | 50.176
      |            \- Stencil Mask Subjects                                          | 149    | 19.456
      |              \- WorldGridMaterial SM_Rock                                    | 149    | 19.456
1749  |                \- DrawIndexed(3684)                                          | 149    | 19.456
1774  |             - DrawIndexed(36)                                                | 150    | 30.72
      |           - PerObject SM_Rock_19                                             | 151    | 37.888
1787  |            \- DrawIndexed(36)                                                | 151    | 13.312
1803  |             - DrawIndexed(36)                                                | 152    | 24.576
1807  |           - API Calls                                                        | 153    | 0.00
      |         - InjectTranslucentVolume                                            | 154    | 46.08
1830  |          \- DrawInstanced(4, 7)                                              | 154    | 29.696
1841  |           - DrawInstanced(4, 3)                                              | 155    | 16.384
      |         - BeginRenderingSceneColor                                           | 156    | 0.00
1847  |          \- API Calls                                                        | 156    | 0.00
      |         - StandardDeferredLighting                                           | 157    | 760.832
1874  |          \- DrawIndexed(1296)                                                | 157    | 760.832

      |       - NewWorld.PointLight_stationary2                                      | 158    | 1102.752
                ...

      |   - FilterTranslucentVolume 64x64x64 Cascades:2                              | 170    | 558.08
2075  |    \- DrawInstanced(4, 64)                                                   | 170    | 280.576
2087  |     - DrawInstanced(4, 64)                                                   | 171    | 277.504
2090  |     - API Calls                                                              | 172    | 0.00
```

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/ShadowedLights.jpg" width="800"  style="display:block; margin:auto;">

As a final step the translucency lighting volumes (for both cascades) are filtered in **FilterTranslucentVolume** to suppress aliasing when lighting translucent props/effects.

# Reflection
Here Unreal calculates and applies reflections. ScreenSpaceReflections takes Hi-Z buffer to speed up raymarching intersection calculation - for rougher surface using low mip and for more reflective surface using high mip for better reflection.

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/ScreenSpaceReflections.jpg" width="400"  style="display:block; margin:auto;">

Then ReflectionEnvironmentAndSky takes environment reflection captures into consideration. The environment reflection probes are generated during game startup and they only capture static geometry. The captured reflections are stored in a mipmapped cubemap per probe.

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/ReflectionEnvironmentAndSky.jpg" width="400"  style="display:block; margin:auto;">

```c
      |   - ScreenSpaceReflections 1916x1014                                         | 173    | 712.704
2095  |    \- ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)          | 173    | 13.312
2123  |     - DrawIndexed(3)                                                         | 174    | 699.392
2126  |     - API Calls                                                              | 175    | 0.00

      |   - ReflectionEnvironmentAndSky                                              | 176    | 2372.64
      |    \- BeginRenderingSceneColor                                               | 176    | 0.00
2133  |      \- API Calls                                                            | 176    | 0.00
2163  |     - DrawIndexed(6)                                                         | 177    | 2372.64
      |     - ResolveSceneColor                                                      | 178    | 0.00
2168  |      \- API Calls                                                            | 178    | 0.00

2170  |   - ResolveSceneColor                                                        | 179    |
2172  |   - CompositionAfterLighting                                                 | 179    |
      |   - BeginRenderingSceneColor                                                 | 179    | 0.00
2179  |    \- API Calls                                                              | 179    | 0.00
```

# Atmosphere and Fog
<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/ExponentialHeightFog.jpg" width="400"  style="display:block; margin:auto;">

```c
      |   - Atmosphere 1916x1014                                                     | 180    | 561.152
2200  |    \- DrawIndexed(6)                                                         | 180    | 561.152
2202  |     - API Calls                                                              | 181    | 0.00
      |   - BeginRenderingSceneColor                                                 | 182    | 0.00
2208  |    \- API Calls                                                              | 182    | 0.00

      |   - ExponentialHeightFog 1916x1014                                           | 183    | 972.768
2229  |    \- DrawIndexed(6)                                                         | 183    | 972.768
2231  |     - API Calls                                                              | 184    | 0.00

      |   - GPUParticles_PostRenderOpaque                                            | 185    | 23.52
      |    \- GPUParticles_SimulateAndClear                                          | 185    | 16.352
      |      \- ParticleSimulationCommands                                           | 185    | 16.352
      |        \- ParticleSimulation                                                 | 185    | 16.352
2275  |          \- DrawIndexed(48)                                                  | 185    | 16.352
2281  |           - API Calls                                                        | 186    | 0.00
      |     - ParticleSimulation                                                     | 187    | 7.168
2298  |      \- DrawIndexed(48)                                                      | 187    | 7.168
2306  |       - API Calls                                                            | 188    | 0.00
```
Note that here we have another particle pass GPUParticles_PostRenderOpaque.


# Translucency
From here the engine finally starts to process the translucent objects in my case the 2 sculptures and the fire particles.

Transparent props are affected by the **local and directional lights, environmental reflections, fog etc**. By default, the renderer uses a high quality shader to render

1. transparent props which samples
2. the atmospheric simulation precomputed textures,
3. baked lightmap data,
4. the translucency lighting volumes which contain lighting from the directional and
5. local lights and
6. the reflection probe cubemaps and uses them to calculate lighting.

Lots of previous rendering data are needed at this stage for translucency rendering which is why translucency is expensive.

```c
      |   - Translucency                                                             | 189    | 158.752
      |    \- BeginRenderingSceneColor                                               | 189    | 0.00
2316  |      \- API Calls                                                            | 189    | 0.00
2328  |     - Draw(4)                                                                | 190    | 52.224
      |     - M_StatueGlass None 2 instances                                         | 191    | 106.528
2378  |      \- DrawIndexedInstanced(9768, 2)                                        | 191    | 106.528
2382  |       - API Calls                                                            | 192    | 0.00

      |   - Translucency                                                             | 193    | 599.04
      |    \- BeginSeparateTranslucency                                              | 193    | 12.288
2392  |      \- ClearRenderTargetView(0.000000, 0.000000, 0.000000, 1.000000)        | 193    | 12.288
2395  |       - API Calls                                                            | 194    | 0.00
      |     - M_Fire_SubUV P_Fire 4 instances                                        | 195    | 71.712
2421  |      \- DrawIndexedInstanced(6, 4)                                           | 195    | 71.712
      |     - M_Fire_SubUV P_Fire 5 instances                                        | 196    | 51.168
2428  |      \- DrawIndexedInstanced(6, 5)                                           | 196    | 51.168
      |     - M_smoke_subUV P_Fire 5 instances                                       | 197    | 304.128
2451  |      \- DrawIndexedInstanced(6, 5)                                           | 197    | 304.128
      |     - M_Radial_Gradient P_Fire 5 instances                                   | 198    | 50.176
2471  |      \- DrawIndexedInstanced(96, 5)                                          | 198    | 50.176
      |     - M_Radial_Gradient P_Fire 2 instances                                   | 199    | 42.976
2478  |      \- DrawIndexedInstanced(96, 2)                                          | 199    | 42.976
      |     - M_Heat_Distortion P_Fire 5 instances                                   | 200    | 66.592
2494  |      \- DrawIndexedInstanced(6, 5)                                           | 200    | 66.592
2498  |       - API Calls                                                            | 201    | 0.00
2499  |     - ResolveSeparateTranslucency                                            | 202    |
```

<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/Translucency.gif" width="400"  style="display:block; margin:auto;">

# Distortion
Both transparent props and particles (that are set to refract) are rendered again to write out a full resolution buffer with the **distortion vectors** that will later be used to calculate refractio. The stencil buffer is also active during that pass to mark the pixels that need refracting.

```c
      |   - Distortion                                                               | 202    | 204.864
      |    \- DistortionAccum                                                        | 202    | 148.544
2508  |      \- ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)        | 202    | 6.112
      |       - M_StatueGlass SM_Statue                                              | 203    | 38.944
2537  |        \- DrawIndexed(9768)                                                  | 203    | 38.944
      |       - M_StatueGlass SM_Statue                                              | 204    | 29.728
2541  |        \- DrawIndexed(9768)                                                  | 204    | 29.728
      |       - M_Heat_Distortion P_Fire 5 instances                                 | 205    | 73.76
2559  |        \- DrawIndexedInstanced(6, 5)                                         | 205    | 73.76
      |     - DistortionApply                                                        | 206    | 56.32
2585  |      \- DrawIndexed(3)                                                       | 206    | 40.96
2601  |       - DrawIndexed(3)                                                       | 207    | 15.36
2603  |       - API Calls                                                            | 208    | 0.00

      |   - ResolveSceneColor                                                        | 209    | 0.032
2608  |    \- API Calls                                                              | 209    | 0.032

```
<img src="{{ site.url }}/images/2019-06-20-unreal-frame-breakdown-part-1/Distortion.jpg" width="400"  style="display:block; margin:auto;">

# PostProcessing
At this stage the renderer applies temporal antialiasing, motion blur, auto exposure calculations, bloom and tonemapping etc. to the main rendertarget.
```c
      |   - PostProcessing                                                           | 210    | 3801.056
      |    \- BokehDOFRecombine#2 1916x1014                                          | 210    | 489.44
2627  |      \- Draw(10)                                                             | 210    | 8.192
2644  |       - DrawIndexed(3)                                                       | 211    | 481.248
2646  |       - API Calls                                                            | 212    | 0.00
      |     - TAA Main PS 1916x1014                                                  | 213    | 1119.232
2677  |      \- DrawIndexed(3)                                                       | 213    | 1039.36
2689  |       - DrawIndexed(3)                                                       | 214    | 79.872
2691  |       - API Calls                                                            | 215    | 0.00
      |     - VelocityFlattenCS 1916x1014                                            | 216    | 562.176
2701  |      \- Dispatch(120, 64, 1)                                                 | 216    | 562.176
2705  |       - API Calls                                                            | 217    | 0.00
      |     - VelocityGatherCS 1916x1014                                             | 218    | 44.032
2713  |      \- Dispatch(8, 4, 1)                                                    | 218    | 44.032
2716  |       - API Calls                                                            | 219    | 0.00
      |     - MotionBlur 1916x1014                                                   | 220    | 348.128
2735  |      \- DrawIndexed(3)                                                       | 220    | 348.128
      |     - Downsample 958x507                                                     | 221    | 156.672
2741  |      \- ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)        | 221    | 5.088
2752  |       - DrawIndexed(3)                                                       | 222    | 151.584
      |     - PostProcessHistogram                                                   | 223    | 137.216
2760  |      \- Dispatch(15, 16, 1)                                                  | 223    | 137.216
2763  |       - API Calls                                                            | 224    | 0.00
      |     - PostProcessHistogramReduce                                             | 225    | 37.888
2777  |      \- DrawIndexed(3)                                                       | 225    | 37.888
      |     - PostProcessEyeAdaptation                                               | 226    | 35.808
2791  |      \- DrawIndexed(3)                                                       | 226    | 35.808
      |     - Downsample 479x254                                                     | 227    | 17.408
2797  |      \- ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)        | 227    | 0.992
2807  |       - DrawIndexed(3)                                                       | 228    | 16.416
            ...
      |     - PostProcessWeightedSampleSum#32Horizontal 15x16 in 15x16               | 237    | 20.448
2880  |      \- DrawIndexed(3)                                                       | 237    | 20.448
      |     - PostProcessWeightedSampleSum#32Vertical 30x16 in 30x16                 | 238    | 14.336
2894  |      \- DrawIndexed(3)                                                       | 238    | 14.336
            ...
      |     - PostProcessCombineLUTs [1] 32x32x32                                    | 249    | 65.536
3072  |      \- DrawInstanced(4, 32)                                                 | 249    | 65.536
      |     - Tonemapper(PS GammaOnly=0 HandleScreenPercentage=0) 1916x1014          | 250    | 383.04
3078  |      \- ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)        | 250    | 8.224
3104  |       - DrawIndexed(3)                                                       | 251    | 374.816

3107  |       - End of Frame                                                         | 252    | 0.00
```

# Wrap up
Following the post [How Unreal Render One Frame](https://interplayoflight.wordpress.com/2017/10/25/how-unreal-renders-a-frame/) I set up my own scene and did this frame capture with RenderDoc, just trying to varify the results for future reference. I will keep coming back here to add more of my own insights about the rendering pipeline.

END
