---
title: "Understand ListView of Editor Utility Widget"
layout: post
image: 2023-04-01-unreal-utility-widget-list-view\cover.png
---

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\cover.png" style="display:block; margin:auto;">


---

# What is List View

A virtualized list that allows up to thousands of **items** to be displayed.

There is an important distinction between the concepts of "**Item**" and "**Entry**". The list itself stores a list of N **items**, but only creates as many **entry widgets** as can fit on screen.

> For example, a scrolling ListView of 200 items with 5 currently visible will only have created 5 entry widgets.

To make a widget usable as an **entry** in a ListView, it must inherit from the **IUserObjectListEntry interface**.

To add an item with custom properties, we need:

- An ```Editor Utility Widget``` with a List View widget
- An ```Object``` blueprint to store the data of the list item (EntryData)
- A ```User Widget``` blueprint to represent of list item (EntryWidget)

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_4.png" style="display:block; margin:auto;">

The idea behind this setup is most likely to **keep data and visual presentation separated**.

### Entry Data

The entry data object's purpose is only to store data. Create it as a Blueprint class that inherits from **Object**.

We will revisit it later.

### Entry Widget

The entry widget will be the visual representation of the list item data. Create it as a Blueprint class that inherits from **User Widget**.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_5.png" style="display:block; margin:auto;">

Then open the blueprint and switch to graph mode, click class setting and Implement the **User Object List Entry** interface.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_3.png" style="display:block; margin:auto;">

After that a few more events will be available on the side in the interface section.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_7.png" style="display:block; margin:auto;">

### Editor Utility Widget

Inside EUW, add a List View and scroll down to add List Entries.

It should show up in the drop-down if the entry widget is properly setup and implemented the required interface mentioned above.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_9.png" style="display:block; margin:auto;">

Note that by default there will be 5 preview entries inside the widget editor but once compiled no entries will show up in the EUW. More following steps are required.

Inside the blueprint graph, use **construct object from class** node to construct entry data and add to list view as item. Use for loop to add more.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_12_1.png" style="display:block; margin:auto;">

Compile the EUW and run, 5 entries will show up as a list.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_14.png" style="display:block; margin:auto;">

### Presenting the data in widget

What if we want to dynamically to pass data to entry widget to present?

We have to utilize both **Entry Data** blueprint and  
**UserObjectListEntry interfac**e implemented in the **Entry Widget**.

First add the data as a variable to **Entry Data** object blueprint. Make sure it is **public, Instance Editable and Expose on Spawn**.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_10.png" style="display:block; margin:auto;">

Back in Entry Widget blueprint, right click to implement the interface event **Event on List Item Object Set** and pass the data from inside of variable to the widget to display (TextBlock for example here)

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_8.png" style="display:block; margin:auto;">

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_11.png" style="display:block; margin:auto;">

Back in EUW, we can pass dynamic data (index here for example) during constructing Entry Data. Right click to refresh the construct node if the **index** input is not showing.

<img src="{{ site.url }}/images\2023-04-01-unreal-utility-widget-list-view\Screenshot_13.png" style="display:block; margin:auto;">
