---
title: Compute APIs
date: 2018-12-13 15:51:32
categories:
  - Compute
tags:
  - Compute
---

Compute APIs: CUDA, HIP, OpenCL.

<!--more-->

# Compute APIs Table

Term                    | CUDA                  | HIP                   | OpenCL
----------------------- | --------------------- | --------------------- | ----------------------
**Device**              | int deviceId          | int deviceId          | cl_device
**Queue**               | cudaStream_t          | hipStream_t           | cl_command_queue
**Event**               | cudaEvent_t           | hipEvent_t            | cl_event
**Memory**              | void *                | void *                | cl_mem
                        | grid                  | grid                  | NDRange
                        | block                 | block                 | work-group
                        | thread                | thread                | work-item
                        | warp                  | warp                  | sub-group
                        |                       |                       |
**Grid dim**            | gridDim.x             | hipGridDim_x          | get_global_size(0)
**Block dim**           | blockDim.x            | hipBlockDim_x         | get_local_size(0)
**Block index**         | blockIdx.x            | hipBlockIdx_x         | get_group_id(0)
**Thread index**        | threadIdx.x           | hipThreadIdx_x        | get_local_id(0)
                        |                       |                       |
**Device Function**     | \_\_device\_\_        | \_\_device\_\_        | In device Compilation
**Host Function**       | \_\_host\_\_          | \_\_host\_\_          | In host Compilation
**Host + Device Function**  | \_\_device\_\_ \_\_host\_\_ | \_\_device\_\_ \_\_host\_\_  | No
**Kernel Launch**       | <<< >>>               | hipLaunchKernel       | clEnqueueNDRangeKernel
                        |                       |                       |
**Global Memory**       | \_\_global\_\_        | \_\_global\_\_        | \_\_global
**Group Memory**        | \_\_shared\_\_        | \_\_shared\_\_        | \_\_local
**Constant**            | \_\_constant\_\_      | \_\_constant\_\_      | \_\_constant


* Reference:
  - [Table Comparing Syntax For Different Compute APIs](https://rocm-documentation.readthedocs.io/en/latest/Programming_Guides/Programming-Guides.html?highlight=hip_porting_driver_api#table-comparing-syntax-for-different-compute-apis)
