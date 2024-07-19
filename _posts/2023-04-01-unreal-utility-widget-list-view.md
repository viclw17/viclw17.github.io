---
title: "Understand ListView of Editor Utility Widget"
layout: post
image: 2023-04-01-unreal-utility-widget-list-view/cover.png
---

<!-- <img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\cover.png" style="display:block; margin:auto;"> -->

**List View** is a useful type of widget available in **Editor Utility Widget** Blueprint (EUW). During editor tooling in Unreal Engine, there are many cases require querying assets or actors and display them as a list. 

One of the recent tools I worked on will list out all the cinematic cameras in the level as a list view widget (a subset of tools for our Virtual Production workflow). From it we can snap to the camera view with options to lock and unlock the viewport control. 

However, the setup of List View is not very striaightforward and addtional cares must be taken even for the minimal functionality. Most of the documentations are scattered around community pages so here I will document my practice.

---

# What is List View

A virtualized list that allows up to thousands of **items** to be displayed.

There is an important distinction between the concepts of "**Item**" and "**Entry**". The list itself stores a list of N **items**, but only creates as many **entry widgets** as can fit on screen.

> For example, a scrolling ListView of 200 items with 5 currently visible will only have created 5 entry widgets.

To add an item with custom properties, we need:

- An ```Editor Utility Widget``` with a List View widget
- An ```Object``` blueprint to store the data of the list item (EntryData)
- A ```User Widget``` blueprint to represent of list item (EntryWidget)

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_4.png" style="display:block; margin:auto;">

To make a widget usable as an **entry** in a ListView, it must inherit from the **UserObjectListEntry interface**. More details will follow.

> The idea behind this setup is to **keep data and visual presentation separated**.

# Entry Data

The entry data object's purpose is only to store data. 

Create it as a Blueprint class that inherits from **Object**.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_5_0.png" style="display:block; margin:auto;">

We will revisit it later.

# Entry Widget

The entry widget will be the visual representation of the list item data. Create it as a Blueprint class that inherits from **User Widget**.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_5.png" style="display:block; margin:auto;">

Then open the blueprint and switch to graph mode, click **class setting** and Implement the **User Object List Entry** interface.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_3.png" style="display:block; margin:auto;">

After that a few more events will be available on the side in the interface section.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_7.png" style="display:block; margin:auto;">

# Editor Utility Widget

## Assign List Entry Widget

Inside EUW, add a List View and scroll down to add **List Entries**.

**EntryWidget** we just created should show up in the drop-down if it is properly setup and **implemented the required interface** mentioned above.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_9.png" style="display:block; margin:auto;">

Note that by default there will be 5 preview entries inside the widget editor but once compiled no entries will show up in the EUW. More following steps are required.

## Construct List Entry Data

Inside the blueprint graph, use **construct object from class** node to construct entry data and add to list view as item. Use for loop to add more.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_12_1.png" style="display:block; margin:auto;">

Compile the EUW and run, 5 entries will show up as a list.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_14.png" style="display:block; margin:auto;">

However, note that at this point all the entry widgets are showing the default data (default text in text block here). We still need one more step to pass entry data to the widget to present.

# Presenting data in widget

What if we want to pass custom data to entry widget to present?

We have to utilize both **Entry Data** blueprint and  
**UserObjectListEntry interface** implemented in the **Entry Widget**.

First, add the data as a variable to **Entry Data** object blueprint. Make sure it is **public, Instance Editable and Expose on Spawn**.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_10.png" style="display:block; margin:auto;">

Back in Entry Widget blueprint, right click to implement the interface event **Event on List Item Object Set** and pass the data from inside of variable to the widget to display (TextBlock for example here).

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_8.png" style="display:block; margin:auto;">

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_11.png" style="display:block; margin:auto;">

Back in EUW, we can pass dynamic data (index here for example) during constructing Entry Data. Right click to refresh the construct node if the **index** input is not showing.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_13.png" style="display:block; margin:auto;">

This the minimal workflow to setup and use the List Widget in EUW. More notes will be added here for more conplex usage of the feature.

TBC