---
layout: post
title: MatCap Shader Showcase
description: "Showcasing the infinite possibility of MatCap shader."
modified: 2016-05-01
tags: [Unity Shader]
image:
  feature: matcap.png
  credit:
  creditlink:
---

MatCap (Material Capture) shader, for displaying objects with reflective materials with uniform surface colouring, like Zbrush or Mudbox can. It uses an image of a sphere as a view-space environment map. It's very cheap, and looks great when the camera doesn't rotate.

**Resources:**

* [Explaination of application in Zbrush](http://docs.pixologic.com/user-guide/materials-lights-rendering/materials/matcap/matcap-basics/)
* [Paper: The Lit Sphere: A Model for Capturing NPR Shading from Art](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.29.1869&rep=rep1&type=pdf)


This is a simple MatCap shader showcasing application I made by Unity:

* Scroll mouse wheel to zoom;
* click and drag to change view angle;
* click top arrow to review the MatCap texture panel and try different effect.

<!-- <iframe src="{{ site.url }}/app/MatCap/MatCap.html" width="600" height="650" scrolling="no" frameborder="0" align="middle"> -->
<iframe src="{{ site.url }}/app/MatCap_webgl/index.html" width="600" height="650" scrolling="no" frameborder="0" align="middle">
</iframe>

<br>

This is the shader code of a typical MatCap shader. It is extremely straightforward on the theory and easy to implement.


{% highlight c %}
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
{% endhighlight %}

TBC
