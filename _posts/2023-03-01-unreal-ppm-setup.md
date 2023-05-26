---
title: "Access Post Process Materials"
layout: post
image: 2023-03-01-unreal-ppm-setup\5.png
---

Manipulating Post Processing Materials (PPM) and Post Processing Volume actor (PPV) in blueprint utility scripts could be tricky sometimes. Here I documented some tips for future reference.

# Get PPM properties

Get a PPV actor then get its Settings (type is **Post Process Settings Structure**). We can **Get All Actors Of Class** then pick the 1st actor in the array (assuming there is only 1 PPV in the level), or use **Get Actor Of Class** which by default picking the 1st actor.

<img src="{{ site.url }}/images\2023-03-01-unreal-ppm-setup\1.png" style="display:block; margin:auto;">

Select **Settings** node and break the structure, then enable the property pins that we want to access in the details panel.

<img src="{{ site.url }}/images\2023-03-01-unreal-ppm-setup\2.png" style="display:block; margin:auto;">



The **Post Process Materials** property itself is another structure (type **Weighted Blendables Structure**) which contains an array of **Weighted Blendable Structures**.

> Weighted Blendable Structures - Allow custom post process materials to be defined using a Material Instance with the same Material as its parent to allow blending. For materials, this needs to be in the PostProcess domain type.

Break the structure and get the first blendable structure, and break again to access the blendable properties - **Weight** and **Object**.

<img src="{{ site.url }}/images\2023-03-01-unreal-ppm-setup\3.png" style="display:block; margin:auto;">



## Cast object to MaterialInstance

To access PPM properties, we can cast object to **MaterialInstance**.

Get **Scalar Param Values** (structures) of the Material Instance and for each one break the structure. Inside we can find another structure called **Material Parameter Info** and the **Parameter Value**. 

Break the Material Parameter Info to get the **Name** property.

Now we get both the parameter name and the parameter value.

<img src="{{ site.url }}/images\2023-03-01-unreal-ppm-setup\4.png" style="display:block; margin:auto;">



## Cast object to MaterialInstanceConstant

Later research found if cast the object to **MaterialInstanceConstant**, we can get access to the direct functions to do so. 

Those functions are all from **Material Editing Library** that only applicable for **MaterialInstanceConstant** type of objects.

<img src="{{ site.url }}/images\2023-03-01-unreal-ppm-setup\5.png" style="display:block; margin:auto;">

<img src="{{ site.url }}/images\2023-03-01-unreal-ppm-setup\6.png" style="display:block; margin:auto;">



# Set PPM properties

There seems no direct approach to set material instance parameters to **MaterialInstance** object unless we cast the object to **MaterialInstanceConstant**.

<img src="{{ site.url }}/images\2023-03-01-unreal-ppm-setup\7.png" style="display:block; margin:auto;">



# Assign material asset to PPV

Just work backwards to build up the weighted blendables array and feed it into the post processing settings structure.

<img src="{{ site.url }}/images\2023-03-01-unreal-ppm-setup\8.png" style="display:block; margin:auto;">

TBC for more notes.