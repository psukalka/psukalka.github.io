+++
date = '2025-05-09T19:51:31+05:30'
draft = false
title = 'Vector Graphics'
+++
If you zoom in on any image on your phone, it starts breaking after 2x - 3x zoom. You can see square pixel blocks. But what happens when you zoom in on google map ? Despite how much you zoom (till you hit last block) it doesn't seem to break. I observed similar thing with Figma. 

What is the magic here ? Why doesn't it break ? Let's try to reverse engineer the problem. How would I achieve it if I want to create such google map like functionality. 

A simple solution can be to generate a highly detailed image and then share it. 
But there is a problem with this. With detailing, size of image increases. With size, loading time will also increase. 
But that doesn't seem to be the case when you load a map. On the contrary, when you zoom it seems to load more details. 
So, does it send low resolution time at first and then send other images afterward ? 
But how would it exactly fit the new image inside the previous zoomed image ? 
Also, road conditions can keep varying for each person, so does it keep generating billions of images every second ? 
That doesn't seem like a scalable solution. 

Well, if you notice there is something peculiar about maps or engineering designs. They involve mostly geometric shapes. Lines, circles, curves, polygons, ... And since these are geometric shapes, we can represent them concisely with mathematical formulas. 
So, instead of having a `1000x1000` pixel circle having color in each pixel, we can represent it simply with a center, radius and color to fill. Thus, a 4MB `jpg` circle can be stored in 1kB `svg` file. 
Only catch is the client should know how to draw image using the mathematical formula. WebGL is used in browsers to render vector graphics. 

This is `Vector graphics`. Instead of storing what each pixel should show, the data is encapsulated in mathematical shapes that client can understand. So, despite of how much you zoom, image will never break. On the contrary, when you zoom in, the client (mobile phone OS) draws only that portion of design. A vector graphic image does not break even at infinite zoom.

The way of storing data in each pixel is called `Raster Graphics`. This is the most common way of storing images like jpg, png, ... Vector graphic images are stored as .svg (Scalable Vector Graphics) files.

Filters, shapes, animations can be easily represented with vector graphics using minimal memory. 
Another issue with raster graphics is that they are not editing friendly. You never edit only one pixel, you would redraw some shape or give some new color to certain group of pixels. This can never be precise. So, with multiple editing there is loss of quality. 
On the contrary, vector graphics are very editing friendly. Each design is a group of elements. You can remove or edit any element precisely. 

Well, if everything is so hunky-dory with vector graphics then why don't we use it for everything ? Why use raster grahics at all ? 

As discussed above, vector graphics need mathematical formulas. So, only geometric shapes can be precisely drawn with it. If you try to draw a photo with svg, its size will increase rather than decreasing. 

Also, how much memory will a svg take isn't known beforehand. An (uncompressed) 4MB jpg image will take around that much memory but a kB of svg can take MBs of memory at run time depending on complexity of elements used. Ex. rendering filters (like gaussian blur) at runtime take more memory (in MBs). 

An interesting case is of [Billion Laughs](https://en.wikipedia.org/wiki/Billion_laughs_attack) where a few kB image can explode to GBs of memory usage. 

A sample SVG can look like: 
```
<!DOCTYPE html>
<html>
<body>
  <svg width="150" height="150" xmlns="http://www.w3.org/2000/svg">
    <rect width="100%" height="100%" fill="green" />
    <circle cx="75" cy="75" r="40" fill="yellow" />
    <text x="75" y="75" font-size="30" text-anchor="middle" fill="red">PAVAN</text>
  </svg>
</body>
</html>
```
It is just xml rather than pixel data. 

Engineering designing softwares like AutoCAD, Figma, Adobe Illustrator, ... that need precise geometric shapes and multiple editing rely on vector graphics. Same is the case with google maps where roads and houses are shapes with traffic being indicated by fill color. 

So, vector graphics is not an alternative to raster graphics. It is complimentary. The cases which have geometric patterns, animations can be compressed and displayed with higher precision using vector graphics. 