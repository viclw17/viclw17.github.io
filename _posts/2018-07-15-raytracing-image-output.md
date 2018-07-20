---
title: Raytracing - Image Output
date: 2018-07-15
tags:
- Computer Graphics
- Raytracing
- Ray Tracing in One Weekend
- PBR
---
<img src="{{ site.url }}/images/raytracing-antialiasing-1.jpg" width="640"  style="display:block; margin:auto;">
<figcaption style="text-align: center;">The "Hello World!" of Computer Graphics.</figcaption>
<br />
Kickstarter of book [Ray Tracing in One Weekend](http://in1weekend.blogspot.com/2016/01/ray-tracing-in-one-weekend.html) (RTIOW). PPM image format, C++ file output, and resources.

# Image Output
There are lots of file formats to choose to pack raytracing results, and the one RTIOW recommend is PPM format.

## PPM Format

PPM aka the **portable pixmap format**, along with the portable graymap format (PGM) and the portable bitmap format (PBM) are image file formats belong to Netpbm formats family, designed to be easily exchanged between platforms.
```
P3
3 2
255
# The part above is the header
# "P3" means this is a RGB color image in ASCII
# "3 2" is the width and height of the image in pixels
# "255" is the maximum value for each color
# The part below is image data: RGB triplets
255   0   0     0 255   0     0   0 255
255 255   0   255 255 255     0   0   0
```
save the code with ```.ppm``` extension and open in software like [XnViewer](https://www.xnview.com/en/) :

<img src="https://upload.wikimedia.org/wikipedia/commons/5/57/Tiny6pixel.png" width="320"  style="display:block; margin:auto;">

## C++ File Output
We can just use standard input/output stream objects from ```<iostream>``` to output results and copy them from command line and save as ```.ppm``` file, or we can write files directly with library ```<fstream>```.

``` c
#include <iostream>
#include <fstream>

using namespace std;

int main()
{
    // Resolution 200x100 pixels
    int nx = 200;
    int ny = 100;

    ofstream outfile( "render.ppm", ios_base::out);
    // Output to .ppm file
    outfile << "P3\n" << nx << " " << ny << "\n255\n";
    // Output to command line
    std::cout << "P3\n" << nx << " " << ny << "\n255\n";

    // Draw image pixels from top to bottom, left to right
    for (int j = ny-1; j >= 0; j--)
    {
        for (int i = 0; i < nx; i++)
        {
            float r = float(i) / float(nx);
            float g = float(j) / float(ny);
            float b = 0.2;
            // Get rgb values within range (0.0, 1.0)
            // Cast to integers and map to (0, 255)
            int ir = int (255.99*r);
            int ig = int (255.99*g);
            int ib = int (255.99*b);

            outfile << ir << " " << ig << " " << ib << "\n";
            std::cout << ir << " " << ig << " " << ib << "\n";
        }
    }
}
```
Output image is the one on the top of the page.

# Other Formats
On [Peter Shirley's Graphics Blog](http://psgraphics.blogspot.com/2015/06/a-small-image-io-library-stbimageh.html) he introduced a small all-in-one image IO library: [stb_image.h](https://github.com/nothings/stb/blob/master/stb_image.h) to output more common image formats.
JPEG baseline & progressive (12 bpc/arithmetic not supported, same as stock IJG lib)
- PNG 1/2/4/8/16-bit-per-channel
- TGA (not sure what subset, if a subset)
- BMP non-1bpp, non-RLE
- PSD (composited view only, no extra channels, 8/16 bit-per-channel)
- GIF (*comp always reports as 4-channel)
- HDR (radiance rgbE format)
- PIC (Softimage PIC)
- PNM (PPM and PGM binary only)
