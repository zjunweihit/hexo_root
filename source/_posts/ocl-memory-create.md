---
title: OpenCL> memory creation
date: 2017-12-26 22:54:31
categories:
  - Compute
tags:
  - OpenCL
  - Compute
---

How to create memory on OpenCL device

<!--more-->

# flags for memory creation
* CL_MEM_USE_HOST_PTR
  - Use host memory directly, which could be accessed by device directly.
  - Host and device could access the memory directly, without cl mapping, `clEnqueueMapBuffer()`
  - Both host and device use the unique host memory.
* CL_MEM_ALLOC_HOST_PTR
  - Allocate memory on device side, like vram
  - Host could access the memory by clEnqueueMapBuffer()
  - If set it only, `host_ptr` must be NULL.
  - If set it with CL_MEM_COPY_HOST_PTR, `host_ptr` must be non-NULL.
  - *Invalid* to use CL_MEM_USE_HOST_PTR and CL_MEM_ALLOC_HOST_PTR.
* CL_MEM_COPY_HOST_PTR
  - Copy memory from host to device, `host_ptr` must be non-NULL.
  - The `host_ptr` is different from copied memory on device

* reference
  - [OpenCL1.2: Increase perofrmance on Intel Processor Graphics](https://software.intel.com/en-us/articles/getting-the-most-from-opencl-12-how-to-increase-performance-by-minimizing-buffer-copies-on-intel-processor-graphics)
