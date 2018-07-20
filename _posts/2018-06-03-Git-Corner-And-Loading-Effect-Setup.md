---
title: Git Corner And Loading Effect Setup
date: 2018-06-03
tags:
- WebDev
- Html
---
<!---
Featured image.
-->
<img src="{{ site.url }}/images/github-corner.jpg" width="640"  style="display:block; margin:auto;">
<br>
<!-- <figcaption style="text-align: center;">First PBR rendering test, looking neat. </figcaption> -->
Note for setting up [GitHub Corners](http://tholman.com/github-corners/) gadget on the web page to link to github repository, and for  formatting my [WebGL page](http://viclw17.github.io/apps/WebGL/MatCap_demo/index.html) with nice loading effect.

For GitHub Corners, big thanks to the author [tholman](https://github.com/tholman/github-corners) . And about the loading effect I am using [PACE](http://github.hubspot.com/pace/docs/welcome/). Here is the code for modifying my WebGL html page to setup those gadgets:

``` html
<html>
...
  <head>
    <title>Unity WebGL Player | MatCap_Unity</title>
    <!-- Pace Loading Tool -->
    <script src="/assets/lib/pace/pace.min.js"></script>
    <link href="/assets/lib/pace/pace-theme-center-atom.min.css" rel="stylesheet">
  </head>
  <!-- Change body background to black -->
  <body style="background: #000">
    <!-- Canvas style setup to center the content, mentioned below -->   
    <canvas>...</canvas>
    <!-- Main Unity WebGL Widget -->
    <script>...</script>
    <!-- Github Corner Code, plz refer to the tool website. -->
    <a href="https://github.com/viclw17/Matcap_Unity" class="github-corner" ...
  </body>
</html>
```


Here is the code to modify the layout to center my WebGL app in the very center of the whole page. Here we have to

- use **absolute positioning**
- specify both **width and height**
- **set the left, right, top and bottom properties to 0**
- let the browser fill the remaining space with the **auto margins**

``` html
<canvas height="600px" width="600px"
    style="
        padding: 0;
        margin: auto;
        display: block;
        position: absolute;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;"
></canvas>
```
