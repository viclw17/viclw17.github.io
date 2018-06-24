---
title: GLSL Practice With Shadertoy
date: 2018-06-12
tags:
- GLSL
- Shader
- Shadertoy
- UV
---
<!---
Featured image. width set to 640 to align with shadertoy
-->
<img src="{{ site.url }}/images/glsl-jupiter.jpg" width="640"  style="display:block; margin:auto;">
<!-- <figcaption style="text-align: center;">First PBR rendering test, looking neat. </figcaption> -->
<br />
Getting started to use [Shadertoy](https://www.shadertoy.com/) to learn and practice GLSL. Here are the first few examples I've been playing around - so much fun! Also got the shaders embedded in my blog page!
Here I documented some of my exploration about the website and some best-practice.
(_I also use [KodeLife](https://hexler.net/software/kodelife/) to work on my shader offline.It is an amazing live shader programming tool and I will document the basic usages of it in next blog post._)

# Shadertoy First Try
## Watercolor Blending
Made a simple shader iterating a combination of sine and cosine functions with UV coordinate to achieve a watercolor blending effect. Shader is very simple but the final looking is very organic.

Use mouse click to interact with the motion.
<br>
<iframe width="640" height="360" frameborder="0" src="https://www.shadertoy.com/embed/lsyfWD?gui=true&t=10&paused=false&muted=false" allowfullscreen style="display:block; margin:auto;"></iframe>
<br>
{% highlight c linenos=table %}
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{   
    float speed = .1;
    float scale = 0.002;
    vec2 p = fragCoord * scale;   
    for(int i=1; i<10; i++){
        p.x+=0.3/float(i)*sin(float(i)*3.*p.y+iTime*speed)+iMouse.x/1000.;
        p.y+=0.3/float(i)*cos(float(i)*3.*p.x+iTime*speed)+iMouse.y/1000.;
    }
    float r=cos(p.x+p.y+1.)*.5+.5;
    float g=sin(p.x+p.y+1.)*.5+.5;
    float b=(sin(p.x+p.y)+cos(p.x+p.y))*.5+.5;
    vec3 color = vec3(r,g,b);
    fragColor = vec4(color,1);
}
{% endhighlight %}

## Jupiter
I then take the watercolor blending shader to the next level - mapping the final color output onto a UV sphere and added scrolling and stretching motion. The final looking is just like a Jupiter planet.

I decide to keep pushing this Jupiter shader and add lighting model and fresnel shading etc. in the future to make it more close to the real look of the planet.
<br>
<iframe width="640" height="360" frameborder="0" src="https://www.shadertoy.com/embed/MdyfWw?gui=true&t=10&paused=false&muted=false" allowfullscreen style="display:block; margin:auto;"></iframe>

<!-- # Shadertoy Basics -->
