# notes

https://fruty.io/2018/03/05/the-rendering-equation-explained/

Numerical Solution

This equation belongs to a wider category of equations, called “**Fredholm equations of the second kind**”. So far scientists have found analytical solutions to several subcategories of Fredholm equations of the second kind, unfortunately the rendering equation is not one of them, one of the reasons being that the equation is “**infinitely recursive**”: the energy coming from a ray depends on where and how the ray bounced before.

The rendering equation includes an **integral over surface patches around the point of interest**. The equation can be **re-written as an integral over light paths**. Solving it is then all about deciding which light paths you can ignore, or compute differently, to speed up the process. The integral is **highly dimensional (the space of light paths is very large)**, and for such problems **Monte Carlo** methods are a good fit.

Most renderers today (Arnold etc) use some form of Monte Carlo ray tracing. It is important to understand that Monte Carlo methods are **statistical methods which involve random sampling**, of light paths. This means a lot of **random memory access**, which in turns explains why a lot of commercial renderers perform better on **CPUs** (vs GPUs).

However, as always, it is always possible to implement an algorithm in many ways, and some renderers (ex: Octane Render) implement Monte Carlo ray tracing in a GPU-friendly way.


