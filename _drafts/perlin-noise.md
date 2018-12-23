# Smoothstep
https://en.wikipedia.org/wiki/Smoothstep
<!-- <iframe src="https://www.desmos.com/calculator/xszqzoandu?embed" width="500px" height="500px" style="border: 1px solid #ccc" frameborder=0></iframe> -->

In HLSL and GLSL, smoothstep implements the ${S} _{1}(x)$, the cubic Hermite interpolation after clamping:

$$ {smoothstep} (x)=S_{1}(x)={\begin{cases}0&x\leq 0\\3x^{2}-2x^{3}&0\leq x\leq 1\\1&1\leq x\\\end{cases}}$$

Again, assuming that the left edge is 0, the right edge is 1, with the transition between edges taking place where $0 ≤ x ≤ 1$.

A C/C++ example implementation:
```c
float smoothstep(float edge0, float edge1, float x) {
  // Scale, bias and saturate x to 0..1 range
  x = clamp((x - edge0) / (edge1 - edge0), 0.0, 1.0);
  // Evaluate polynomial
  return x * x * (3 - 2 * x);
}

float clamp(float x, float lowerlimit, float upperlimit) {
  if (x < lowerlimit)
    x = lowerlimit;
  if (x > upperlimit)
    x = upperlimit;
  return x;
}
```
