---
title: Infinite Possibility Of MatCap Shader
layout: post
image: 2016-05-01-MatCap-Shader-Showcase/matcap_bunnies.jpg
---

<img src="{{ site.url }}/images/2016-05-01-MatCap-Shader-Showcase/matcap_bunnies.jpg" width="640"  style="display:block; margin:auto;">
<br>
<!-- ![MatCap Bunnies]({{ site.url }}/images/2016-05-01-MatCap-Shader-Showcase/matcap_bunnies.jpg) -->
MatCap (Material Capture) shader uses an image of a sphere as a view-space environment map. It's very cheap and looks great when the camera doesn't rotate. It is widely used in 3D sculpting software (like Zbrush) to preview meshes.

# Reference

* [Explaination of application in Zbrush](http://docs.pixologic.com/user-guide/materials-lights-rendering/materials/matcap/matcap-basics/)
* [Paper: The Lit Sphere: A Model for Capturing NPR Shading from Art](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.29.1869&rep=rep1&type=pdf)

# Demo

<a href="http://viclw17.github.io/apps/WebGL/MatCap_demo/index.html" target = "_blank">This</a> is a simple MatCap shader showcasing application I made by Unity:
* Scroll mouse wheel to zoom;
* click and drag to change view angle;
* click top arrow to review the MatCap texture panel and try different effect.

<!-- Comment 1 -->
<!-- <iframe src="{{ site.url }}/app/WebGL/MatCap_demo/index.html" width="600" height="600" scrolling="no" frameborder="0" align="middle">
</iframe> -->

<!-- Comment 2 -->
<!-- <a href="{{ site.url }}/app/WebGL/MatCap_demo/index.html" target = "_blank">
<img src="{{ site.url }}/images/2016-05-01-MatCap-Shader-Showcase/matcap_demo_image.png" width="400" height="400" style="display:block; margin:auto;">
</a>
<figcaption style="text-align: center;">Press image to run the demo in new page.</figcaption> -->

<!-- Comment 3 -->
<!-- Press [Here]({{ site.url }}/app/WebGL/MatCap_demo/index.html) to run the demo in new page. -->

# Shader Code

This is the shader code of a typical MatCap shader. It is extremely straightforward on the theory and easy to implement.

```c
Shader "MatCap_Victor/Plain"
{
  Properties
  {
    _Color ("Main Color", Color) = (0.5,0.5,0.5,1)
    _MatCap ("MatCap (RGB)", 2D) = "white" {}
  }

  Subshader
  {
    Tags { "RenderType"="Opaque" }

    Pass
    {
      CGPROGRAM
      	#pragma vertex vert
      	#pragma fragment frag
      	#include "UnityCG.cginc"

      	struct v2f
      	{
          float4 pos	: SV_POSITION;
          float2 cap	: TEXCOORD0;
          float3 model_normal : TEXCOORD1;
          float3 world_normal : TEXCOORD2;
          float3 view_normal : TEXCOORD3;
      	};

        v2f vert (appdata_base v)
        {
          v2f o;
          o.pos = mul (UNITY_MATRIX_MVP, v.vertex);

          // this is model normals
          o.model_normal = v.normal;

          // transform normal vectors from model space to world space
          float3 worldNorm =
            normalize(
            	_World2Object[0].xyz * v.normal.x +
            	_World2Object[1].xyz * v.normal.y +
            	_World2Object[2].xyz * v.normal.z
            	);

          // this is world normals
          o.world_normal = worldNorm;

          // transform normal vectors from world space to view space
          float3 viewNorm = mul((float3x3)UNITY_MATRIX_V, worldNorm);
          // or use built-in UNITY_MATRIX_V

          // this is viewspace normals
          o.view_normal = viewNorm;

          // this is in the context of view space
          // get the coordinate on XY plane, ignore z coordinate
          o.cap.xy = viewNorm.xy * 0.5 + 0.5; // clamp (-1,1) to (0, 1)

          return o;
        }

        uniform float4 _Color;
        uniform sampler2D _MatCap;

        float4 frag (v2f i) : COLOR
        {
          float4 mc = tex2D(_MatCap, i.cap);
          return  _Color * mc;
        }
      ENDCG
    }
  }
  Fallback "VertexLit"
}
```

TBC
