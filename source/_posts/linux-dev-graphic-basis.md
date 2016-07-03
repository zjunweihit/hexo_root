---
title: Graphics Basis
date: 2016-07-03 19:12:16
categories:
  - linux-dev
  - Graphics
tags:
  - linux-dev-misc
---

List the basic graphics knowledge

<!--more-->

# 3D modeling
In 3D computer graphics, 3D modeling (or modelling) is the process of developing a mathematical representation of any three-dimensional surface of an object (either inanimate or living) via specialized software. The product is called a 3D model. It can be displayed as a two-dimensional image through a process called `3D rendering` or used in a computer simulation of physical phenomena. The model can also be physically created using 3D printing devices.

Models may be created automatically or manually. The manual modeling process of preparing geometric data for 3D computer graphics is similar to plastic arts such as sculpting.

3D modeling software is a class of 3D computer graphics software used to produce 3D models. Individual programs of this class are called modeling applications or modelers.

> Referrence
  - [3D modeling](https://en.wikipedia.org/wiki/3D_modeling)

# 3D Rendering
3D rendering is the 3D computer graphics process of automatically converting 3D [wire frame models(线框模型)](http://baike.baidu.com/link?url=NifGBn5OoWPzYA1XosoUvjhkrsYB4mJu5NytWYYvaAid300vcJAW-N70iMEZDZnEPFSXrLKd19dWj9qy0-XWCq) into 2D images with 3D photorealistic effects or non-photorealistic rendering on a computer.

## Rendering methods
**Rendering is the final process of creating the actual 2D image or animation from the prepared scene.** This can be compared to taking a photo or filming the scene after the setup is finished in real life. Several different, and often specialized, rendering methods have been developed. These range from the distinctly non-realistic wireframe rendering through polygon-based rendering, to more advanced techniques such as: scanline rendering, ray tracing, or radiosity. Rendering may take from fractions of a second to days for a single image/frame. In general, different methods are better suited for either photo-realistic rendering, or real-time rendering.

> Referrence
  - [3D rendering](https://en.wikipedia.org/wiki/3D_rendering)

# Graphics pipeline
In 3D computer graphics, the **graphics pipeline** or **rendering pipeline** refers to the sequence of steps used to create a 2D raster representation of a 3D scene.[1] Plainly speaking, once a 3D model has been created, for instance in a video game or any other 3D computer animation, the graphics pipeline is the process of turning that 3D model into what the computer displays.[2] In the early history of 3D computer graphics, fixed purpose hardware was used to speed up the steps of the pipeline through a fixed-function pipeline. Later, the hardware evolved, becoming more general purpose, allowing greater flexibility in graphics rendering as well as more generalized hardware, and allowing the same generalized hardware to perform not only different steps of the pipeline, like in fixed purpose hardware, but even in limited forms of general purpose computing. As the hardware evolved, so did the graphics pipelines, the OpenGL, and DirectX pipelines, but the general concept of the pipeline remains the same.

> Reference
  - [Graphics pipeline](https://en.wikipedia.org/wiki/Graphics_pipeline)
