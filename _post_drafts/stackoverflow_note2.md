#### notes
This is a very good question. There is a common misconception that Monte Carlo, or integration is applied "recursively" on the rendering equation. 

That is not what's happening. 

Numerical integration methods are tailored to problems of the form:

I=∫Ωf(x)dμ(x)≈∑k=0N−1w(xk)f(xk)

Note that this is not the case for the rendering equation. Namely the fact that it is an equation, and **the unknown radiance function L shows up on both sides.** 

In order to apply numerical integration techniques (such as Monte Carlo), one has to bring the rendering equation to the above form. Fortunately this is a well studied problem. 

The rendering equation is a specific case of **a Fredholm integral equation of the second kind**. If one can show that the scattering operator in that equation is a contraction ||T||<1 (which holds if brdfs are energy conserving, there are cases where this is not required but that goes outside the scope of my answer), then we may perform a **Neumann expansion** (https://en.wikipedia.org/wiki/Liouville%E2%80%93Neumann_series).

The commonly used form of the rendering equation is the **solid angle formulation** (here σ is the solid angle measure, σ(ω)=sinθdθdϕ):

L(x,ωo)=Le(x,ωo)+∫Ωf(ωo,x,ωi)Li(x,ωi)cosθidσ(ωi)

In order to perform the expansion, I will rewrite this in the **area formulation** (here μ(x) is the Lebesgue area measure):

L(x1→x0)=Le(x1→x0)+∫Mf(x2→x1→x0)L(x2→x1)cosθx1cosθx2||x2−x1||2V(x2,x1)dμ(x2)

- Where M is the set of all surface points in the scene, 
- V(x,y) is the visibility function which is 1 if there is nothing between x and y and 0 otherwise. 
- If the normals of the surfaces at point x1 and x2 are respectively Nx1 and Nx2, - then cosθx1=Nx1⋅x2−x1||x2−x1|| and cosθx2=Nx2⋅x1−x2||x2−x1||. 
- As for the radiance, L(x2→x1) gives the radiance arriving at x1 from the direction - of x2. 
- And the brdf notation relationship is: f(x2→x1→x0)=f(x1→x0,x1,x1→x2)(that means - ωo=x1→x0 and ωi=x1→x2).

I will rewrite the above for simplicity and conciseness as:

L(x1→x0)=Le(x1→x0)+Lr(x1→x0)

Let us now split the incoming radiance into **direct illumination arriving at x1**
 and **indirect (at least one bounce) illumination**. The direct illumination is obviously due to direct light source rays arriving at x1
:


L(x1→x0)=Le(x1→x0)+∫Mf(x2→x1→x0)(Le(x2→x1)+Lr(x2→x1))cosθx1cosθx2||x2−x1||2V(x2,x1)dμ(x2)=Le(x1→x0)+∫Mf(x2→x1→x0)Le(x2→x1)cosθx1cosθx2||x2−x1||2V(x2,x1)dμ(x2)+

∫Mf(x2→x1→x0)[∫Mf(x3→x2→x1)L(x3→x2)cosθx2

cosθx3||x3−x2||2V(x3,x2)dμ(x3)]cosθx1cosθx2||x2−x1||2V(x2,x1)dμ(x2)


All I did here was separate the sum Le(x2→x1)+Lr(x2→x1)
 into two integrals, and then I additionally **expanded Lr
 into the corresponding integral using the recursive definition**. You can expand this until infinity. 
 
 For conciseness, I will rewrite the integration as an operator: Lr=TL
. Now we can see the relationship to the Neumann expansion. Since the rendering equation can be written as:

L=Le+TL

The solution is formally given as:

(I−T)L=Le
L=(I−T)−1Le

Applying the Neumann expansion yields:

L=∑i=0∞TiLe

Note that each term of the sum is an increasingly dimensional integral. 

The first term is obviously just Le(x1→x0)
, the second term T Le
 is the integral for Le(x2→x2)
 that I wrote when I split Le
 and Lr
, and so on. 

What each term gives you is the energy coming from a light source after i
 bounces. The first term gives you the radiance emitted from x1
 towards x0
 (paths of length 0), the second term gives you the radiance that is due to direct illumination of x1
 scattered towards x0
 (paths of length 1), then you have the radiance due to lights two bounces away (paths of length 2) etc. 
 
 With each bounce each new integral in the sum gains two dimensions (if we are just integrating over incoming directions). I would also like to emphasize that the integration variable x2
 in T Le
 is not the same x2
 as the one in Ti Le
 (this means that in the example with integration, x2
 in the first integral is not the same as in the second one - they are just integration variables). 
 
 Another important fact is that within each sum each path starts at the film x0
 and ends at a light source (Le(x2→x1)
 is non-zero only for points x2
 lying on light sources).

**Now that we have a sum of integrals, we can apply our numerical techniques to each integral to estimate the sum.**

An obvious optimization, is **reusing the samples** used to compute TiLe
 to compute Ti+1Le=T(Ti Le)
. 

**This yields a formulation where we can formally write out the sum of integrals as one integral with integration over all paths that start at the camera film and end at a light source.**

This also illustrates why Monte Carlo is a preferred technique: since we want to estimate an infinite dimensional integral, and Monte Carlo's convergence doesn't depend on the dimensionality unlike standard quadrature rules (there's also the point that it doesn't care about the smoothness of the integrand, but that's a double edged sword).

As you may have noticed, with the **path integration formulation** an exponential path count growth is not required. 

Splitting a path into multiple new ones is a technique known as splitting. While it is beneficial for high energy paths, on average with each bounce the energy gets lower due to attenuation, so this is in most cases counterproductive. 

On the other hand early path termination (for low energy paths) is more often than not beneficial in terms of efficiency (that's simply cutting off the sum at some point and not computing the remaining infinitely many terms, because you deem they have a contribution that is too low), and that's where Russian roulette comes in.

All of this is actually contained in Kajiya's paper on the rendering equation, even though he doesn't go into the details I went. He just refers to Rubinstein's book: https://dl.acm.org/citation.cfm?id=539488 (note - the first edition, later editions do not have the part he is referring to). In this book how to solve Fredholm integrals of the second kind is described (what I explained above, but more formally).

I hope my explanation was useful, and will continue being useful for future readers.