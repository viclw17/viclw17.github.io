

# Lighting

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

      |     - ShadowedLights                                                         | 121    | 5852.064  |   |   |  
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
      |        \- ClearQuad                                                          | 139    | 80.896
1601  |          \- Draw(4)                                                          | 139    | 80.896
1603  |           - API Calls                                                        | 140    | 0.00
      |         - ShadowProjectionOnOpaque                                           | 141    | 512.00
1629  |          \- DrawIndexed(1296)                                                | 141    | 461.824
      |           - InjectTranslucentVolume                                          | 142    | 50.176
1654  |            \- DrawInstanced(4, 7)                                            | 142    | 29.696
1665  |             - DrawInstanced(4, 3)                                            | 143    | 20.48
1667  |             - API Calls                                                      | 144    | 0.00
      |         - BeginRenderingSceneColor                                           | 145    | 0.00
1673  |          \- API Calls                                                        | 145    | 0.00
      |         - StandardDeferredLighting                                           | 146    | 822.272
1701  |          \- DrawIndexed(1296)                                                | 146    | 822.272
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
      |        \- ClearQuad                                                          | 158    | 71.616
1898  |          \- Draw(4)                                                          | 158    | 70.656
1900  |           - API Calls                                                        | 159    | 0.96
      |         - ShadowProjectionOnOpaque                                           | 160    | 84.96
      |          \- PreShadow SM_Rock_19                                             | 160    | 49.152
      |            \- Stencil Mask Subjects                                          | 160    | 19.456
      |              \- WorldGridMaterial SM_Rock                                    | 160    | 19.456
1922  |                \- DrawIndexed(3684)                                          | 160    | 19.456
1947  |             - DrawIndexed(36)                                                | 161    | 29.696
      |           - PerObject SM_Rock_19                                             | 162    | 34.816
1960  |            \- DrawIndexed(36)                                                | 162    | 11.264
1976  |             - DrawIndexed(36)                                                | 163    | 23.552
1980  |           - API Calls                                                        | 164    | 0.992
      |         - InjectTranslucentVolume                                            | 165    | 62.464
2003  |          \- DrawInstanced(4, 7)                                              | 165    | 44.032
2014  |           - DrawInstanced(4, 3)                                              | 166    | 18.432
      |         - BeginRenderingSceneColor                                           | 167    | 0.00
2020  |          \- API Calls                                                        | 167    | 0.00
      |         - StandardDeferredLighting                                           | 168    | 883.712
2047  |          \- DrawIndexed(1296)                                                | 168    | 883.712
2050  |           - API Calls                                                        | 169    | 0.00
```

# Reflection Misc
```c
      |   - FilterTranslucentVolume 64x64x64 Cascades:2                              | 170    | 558.08
2075  |    \- DrawInstanced(4, 64)                                                   | 170    | 280.576
2087  |     - DrawInstanced(4, 64)                                                   | 171    | 277.504
2090  |     - API Calls                                                              | 172    | 0.00

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

# Translucency

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


# PostProcessing
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
      |     - Downsample 240x127                                                     | 229    | 11.296
2813  |      \- ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)        | 229    | 1.056
2822  |       - DrawIndexed(3)                                                       | 230    | 10.24
      |     - Downsample 120x64                                                      | 231    | 8.192
2828  |      \- ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)        | 231    | 0.00
2835  |       - DrawIndexed(3)                                                       | 232    | 8.192
      |     - Downsample 60x32                                                       | 233    | 7.232
2841  |      \- ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)        | 233    | 1.056
2848  |       - DrawIndexed(3)                                                       | 234    | 6.176
      |     - Downsample 30x16                                                       | 235    | 7.168
2854  |      \- ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)        | 235    | 0.00
2861  |       - DrawIndexed(3)                                                       | 236    | 7.168
      |     - PostProcessWeightedSampleSum#32Horizontal 15x16 in 15x16               | 237    | 20.448
2880  |      \- DrawIndexed(3)                                                       | 237    | 20.448
      |     - PostProcessWeightedSampleSum#32Vertical 30x16 in 30x16                 | 238    | 14.336
2894  |      \- DrawIndexed(3)                                                       | 238    | 14.336
      |     - PostProcessWeightedSampleSum#32Horizontal 30x32 in 30x32               | 239    | 14.336
2907  |      \- DrawIndexed(3)                                                       | 239    | 14.336
      |     - PostProcessWeightedSampleSum#32Vertical 60x32 in 60x32                 | 240    | 18.40
2925  |      \- DrawIndexed(3)                                                       | 240    | 18.40
      |     - PostProcessWeightedSampleSum#25Horizontal 120x64 in 120x64             | 241    | 19.456
2941  |      \- DrawIndexed(3)                                                       | 241    | 19.456
      |     - PostProcessWeightedSampleSum#25Vertical 120x64 in 120x64               | 242    | 16.352
2957  |      \- DrawIndexed(3)                                                       | 242    | 16.352
      |     - PostProcessWeightedSampleSum#11Horizontal 240x127 in 240x127           | 243    | 13.312
2974  |      \- DrawIndexed(3)                                                       | 243    | 13.312
      |     - PostProcessWeightedSampleSum#11Vertical 240x127 in 240x127             | 244    | 13.312
2990  |      \- DrawIndexed(3)                                                       | 244    | 13.312
      |     - PostProcessWeightedSampleSum#11Horizontal 479x254 in 479x254           | 245    | 24.576
3005  |      \- DrawIndexed(3)                                                       | 245    | 24.576
      |     - PostProcessWeightedSampleSum#11Vertical 479x254 in 479x254             | 246    | 26.624
3021  |      \- DrawIndexed(3)                                                       | 246    | 26.624
      |     - PostProcessWeightedSampleSum#7Horizontal 958x507 in 958x508            | 247    | 95.20
3037  |      \- DrawIndexed(3)                                                       | 247    | 95.20
      |     - PostProcessWeightedSampleSum#7Vertical 958x507 in 958x508              | 248    | 94.24
3054  |      \- DrawIndexed(3)                                                       | 248    | 94.24
      |     - PostProcessCombineLUTs [1] 32x32x32                                    | 249    | 65.536
3072  |      \- DrawInstanced(4, 32)                                                 | 249    | 65.536
      |     - Tonemapper(PS GammaOnly=0 HandleScreenPercentage=0) 1916x1014          | 250    | 383.04
3078  |      \- ClearRenderTargetView(0.000000, 0.000000, 0.000000, 0.000000)        | 250    | 8.224
3104  |       - DrawIndexed(3)                                                       | 251    | 374.816

3107  |       - End of Frame                                                         | 252    | 0.00
```
