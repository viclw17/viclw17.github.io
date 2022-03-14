---
title: "Unreal Engine Plugin Configuration Quick Note"
layout: post
---

A summery of Unreal Engine Plugin system based on offcial documentation and personal practice.

---
## UE Plugin Usage
In UE4, Plugins are **collections of code and data** that developers can easily enable or disable within the Editor on a per-project basis. 

Plugins can: 
- add runtime gameplay functionality (DLC)
- modify built-in or add new Engine features 
- create new file types (Example?)
- extend the Editor with new menus, tool bar commands, and sub-modes. 

## Plugin Scope Types and Folders
### Engine Plugin
Engine Plugins are available for all projects. Typically, these plugins are created by engine and tools programmers to provide baseline functionality that can be used in multiple projects while being maintained in a single place. This can enable the user to add or override engine features without modifying engine code.

Path: ```/[UE4 Root]/Engine/Plugins/[Plugin Name]/```

### Per-project/Game Plugin
Plugins reside under the Plugins subfolder within your project's directory, and will be detected and loaded at Engine or Editor start-up time.

Path: ```/[Project Root]/Plugins/[Plugin Name]/```

## Plugin Types and Anatomy
### Code Plugin*
Plugins with code will have 
- a ```Source``` folder contains one or more directories with module **source code** for the Plugin.  
- a ```Binaries``` folder that contains **compiled code** for that Plugin
- a ```Intermediate``` folder that contains **temporary build product files** 
- a ```Config``` folder that contains configuration files:
  - Engine plugins: [PluginName]/Config/Base[PluginName].ini
  - Game plugins: [PluginName]/Config/Default[PluginName].ini
- can have their own ```Content``` folder that **contains Asset files specific to that Plugin**.  

Code plugin requires recompile to generate binaries everytime made modification. Unreal will prompt warning during loading if **the enabled plugin**  
- is missing
- is missing dependent plugins
- compiled for different engine version
  
<img src="{{ site.url }}/images/2022-01-20-unreal-engine-plugin-configuration/error.jpg" width="400px;" style="display:block; margin:auto;">

You can choose to recompile from source on spot but usually fail.

To quickly ignore and open the editor, you should:
- **disable the plugins and confirm** on the prompt dialog
- delete the ```Binaries``` and ```Intermediate``` folders and reload project

About the engine version complain, you can use a hack to change the build ID of the plugin. See [Run Sourceless Plugins in a Custom Unreal Build](https://matthewminer.com/2020/09/07/run-sourceless-plugins-in-a-custom-unreal-build).


### Content-only Plugin
Plugin without c++ code but only asset files (```.uasset```) inside plugin ```Content``` folder.

```CanContainContent``` setting within the Plugin's descriptor (```.uplugin``` file) must be set to "true".

> *Plugins do not support their own Derived Data Cache distribution.


## Plugin Descriptor Files (.uplugin)
Plugin descriptors are files that end with ```.uplugin```. The first part of the file name is always the name of your Plugin. Plugin descriptor files are always located in your Plugin's directory, where the Engine will discover them at start-up time.

Plugin descriptors are in the **Json** (JavaScript Object Notation) file format.

### Descriptor File Format
The descriptor file is a JSON-formatted list of variables from the ```FPluginDescriptor``` type. 

<!-- There is one additional field, "**FileVersion**", which is the only required field in the structure. "FileVersion" gives the version of the Plugin descriptor file, and should usually set to the highest version that is allowed by the Engine (currently, this is "3"). Because this version applies to the format of the Plugin Descriptor File, and not the Plugin itself, we do not expect that it will change very frequently, and it should not change with subsequent releases of your Plugin. For maximum compatibility with older versions of the Engine, you can use an older version number, but this is not recommended. -->

For details about the other supported fields, see the [FPluginDescriptor API reference page](https://docs.unrealengine.com/en-US/API/Runtime/Projects/FPluginDescriptor).

### Descriptor File Example
This example plugin descriptor is from the **Engine's UObjectPlugin**.

```
{
    "FileVersion" : 3,
    "Version" : 1,
    "VersionName" : "1.0",
    "FriendlyName" : "UObject Example Plugin",
    "Description" : "An example of a plugin which declares its own UObject type.  This can be used as a starting point when creating your own plugin.",
    "Category" : "Examples",
    "CreatedBy" : "Epic Games, Inc.",
    "CreatedByURL" : "http://epicgames.com",
    "DocsURL" : "",
    "MarketplaceURL" : "",
    "SupportURL" : "",
    "IsBetaVersion" : false,
    "CanContainContent" : false,
    "EnabledByDefault" : true,
    "Installed" : false,
    "Modules" :
    [
        {
            "Name" : "UObjectPlugin",
            "Type" : "Developer",
            "LoadingPhase" : "Default"
        }
    ]
}
```

### Best practice
Important lines:

- FriendlyName: the title of the tool 
- Description: purpose of the tool 
- DocsURL: links to the confluence page/ git repo/ miro board etc. 
- **CanContainContent**: true (will generate content folder for the plugin) 
- IsBetaVersion: false (no need to be true, to avoid annoying pop-up confirm dialog) 
- IsExperimentalVersion: false (no need to be true, to avoid annoying pop-up confirm dialog) 
- **EnabledByDefault**: true (Whether this plugin should be enabled by default for all projects)  
- **Installed**: true (Signifies that the plugin was installed on top of the engine. It will show up in the installed area instead of Others) 

<img src="{{ site.url }}/images/2022-01-20-unreal-engine-plugin-configuration/installed.jpg" style="display:block; margin:auto;">

```Installed: true``` and ```EnabledByDefault: true``` can be useful if you want your plugin to be enabled by default. 

<!-- In this case when you pull down the plugin from the p4 stream and open unreal, you will have this pop-up  -->

<!-- please click update, and a block of config script like below will be generated in your uproject file - which will also prompt you to check out your uproject on p4 because the edit. (better addressed by dev team or p4 admin?)  -->


### Installed vs Enabled (EnabledByDefault) 

- if EnabledByDefault false and Installed is true , plugin won't be enabled
- if EnabledByDefault true, plugin will be enabled even though Installed is false
- if EnabledByDefault omit but Installed is true, plugin will be enabled 

To summerize, **Installed** is only tagging the type of the plugin and **is not the official enable/disable toggle**, however, ```Installed: true``` will trigger plugin to be enabled in uproject, but not the other way around (a enabled plugin may not be installed). 


### Dependencies
If your plugin tool requires other plugin to be enabled, please add "Plugin" section and pay attention to the JSON syntax pitfalls: 

- Be aware to add comma after the last line before "Plugins" section 
- Inside "Plugins" use [] to include multiple plugin dependencies 
- Multiple dependencies use comma , between each {} 
- No more comma after [] 


<!-- When submit your plugin into p4, please select ALL FILES inside the plugin folder and MARK FOR ADD. 
if failed to check in .uplugin file, the plugin will not be successfully loaded into then unreal project.  -->

## Icons
Along with the descriptor file, Plugins need an icon to display in the Editor's Plugin Browser. The image should be **a 128x128 .png file called "Icon128.png"** and kept in the Plugin's ```/Resources/``` directory.




 

 



 

 

 

 