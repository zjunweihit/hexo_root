---
title: OpenCL Overview
date: 2018-12-13 22:17:18
categories:
  - Compute
tags:
  - OpenCL
---

OpenCL Summary

<!--more-->

# Platform model
```
                     +-----------+
                     |           |
               +-----+ Device 1  |
               |     |           |         +------------------------+
  +------+     |     +-----------+         |     Compute Unit 1     |
  |      |     |                           +------------------------+
  |      |     |     +-----------+         |  +--+  +--+      +--+  |
  |      |     |     |           |         |  |  |  |  |      |  |  |
  | host +-----+-----+ Device 2  +-----+---+  |PE|  |PE| ...  |PE|  |
  |      |     |     |           |     |   |  |  |  |  |      |  |  |
  |      |     |     +-----------+     |   |  +--+  +--+      +--+  |
  |      |     |                       |   +------------------ -----+
  +------+     |                       |   ...
               |     ...               |   +------------------------+
               |                       |   |     Compute Unit N     |
               |     +-----------+     |   +------------------------+
               |     |           |     |   |  +--+  +--+      +--+  |
               +-----+ Device N  |     |   |  |  |  |  |      |  |  |
                     |           |     +---+  |PE|  |PE| ...  |PE|  |
                     +-----------+         |  |  |  |  |      |  |  |
                                           |  +--+  +--+      +--+  |
                                           +------------------------+

Computations occurs within PEs

PE -> work-item(threads) -> private memory
CU -> work-group -> local memory
Device -> Command queue, one to one -> global memory

```

# Platform model(HW)

```
                                          +-----------------------------+
                                          |        Shader Engine        |
                                          +-----------------------------+
               +---------------------+    | +------------------------+  |
            +->|        CPC          |--+-+>|     Compute Unit       |  |
            |  +---------------------+  | | +------------------------+  |
            |  |                     |  | | | +--+  +--+  +--+  +--+ |  |
            |  |                     |  | | | |S |  |S |  |S |  |S | |  |
            |  |        (ACE)        |  | | | |I1|  |I1|  |I1|  |I1| |  |
            |  |  MEC X PIPE X QUEUE |  | | | |M6|  |M6|  |M6|  |M6| |  |
            |  |   2     4       8   |  | | | |D |  |D |  |D |  |D | |  |
 +-----+    |  |                     |  | | | +--+  +--+  +--+  +--+ |  |
 |     |    |  +---------------------+  | | +------------------------+  |
 | CPF +----+  +-----+                  | |                             |
 |     |    |  |     |                  | | ...                         |
 +-----+    +->| CPG |                  | | +------------------------+  |
               |     |                  +-+>|     Compute Unit       |  |
               +-----+                    | +------------------------+  |
                                          | | +--+  +--+  +--+  +--+ |  |
                                          | | |S |  |S |  |S |  |S | |  |
                                          | | |I1|  |I1|  |I1|  |I1| |  |
                                          | | |M6|  |M6|  |M6|  |M6| |  |
                                          | | |D |  |D |  |D |  |D | |  |
                                          | | +--+  +--+  +--+  +--+ |  |
                                          | +------------------------+  |
                                          |                             |
                                          +-----------------------------+

amdgpu only owns MEC0, queue[0,1]
(MEC0 is ring->me1: ring->me = mec + 1)

  amdgpu 0000:03:00.0: ring 1(comp_1.0.0) uses VM inv eng 5 on hub 0
  amdgpu 0000:03:00.0: ring 2(comp_1.1.0) uses VM inv eng 6 on hub 0
  amdgpu 0000:03:00.0: ring 3(comp_1.2.0) uses VM inv eng 7 on hub 0
  amdgpu 0000:03:00.0: ring 4(comp_1.3.0) uses VM inv eng 8 on hub 0
  amdgpu 0000:03:00.0: ring 5(comp_1.0.1) uses VM inv eng 9 on hub 0
  amdgpu 0000:03:00.0: ring 6(comp_1.1.1) uses VM inv eng 10 on hub 0
  amdgpu 0000:03:00.0: ring 7(comp_1.2.1) uses VM inv eng 11 on hub 0
  amdgpu 0000:03:00.0: ring 8(comp_1.3.1) uses VM inv eng 12 on hub 0

pre-GCN, VLIW is used in CU as SIMD

* SQ in each CU schedules the execution of waves and processing of work items
```
* AQL: Architecture Queuing Language
  * HSA defines AQL to run CPU and GPU by Apps.
* ACE
  * dispatches one wavefront from a workgroup to CU
* SIMD
  * SIMD-16, includes 16 64-bit ALU, if the package is 32/16/8-bit, it could be several of them in a cycle.
  * each SIMD can work on a separate wavefront
  * several PEs
* wavefront(aka warp for NV)
  * the size of wavefront is (a width of SIMD unit * number of SIMD units) in CU(aligned with CU)
    - 16 * 4 = 64
    - could be 16 ~ 64, also for 1 SMID-16 or 1 CU(4 SIMD-16)
  * Each SIMD unit is assigned its own 40-bit program counter and instruction buffer for 10 wavefronts.
  * the whole CU can have 40 wavefronts in flight, a GPU with 64 CUs, can be working on up to 163,840(64*40*64) work items at a time.
* GDS: synchronize and communicate between tasks running on different SE. Between CUs and SEs.
* LDS: synchronize and communicate inside CU, LDS is 64KB.

# Execution model
* Kernel types
  - OpenCL kernels: functions written in OpenCL C
  - Native kernels: functions created outside of OpenCL and accessed within OpenCL through a function pointer.
    - Functions are defined in host soruce code or exported from a specified library.
    - Option feature
* How a Kernel executes on an OpenCL device
  - work-item: an instance of an excuting kernel
    - running on a PE
    - it can be identified by a global ID or the combination of workgroup ID and local ID
    - each work-item uses the same sequence of the instructions defined by a kernel
    - but the behavrior can vary due to branch statements within the code or data selected through the global ID.
  - work-groups: evenly divides the global size in each dimension
    - concurrently execute on a single CU
  - NDRange: the index space spans an N-dimensioned range
    - N: 1, 2, 3
    - global ID: starts from 0
* Context
  - Devices: OpenCL devices to be used by the host
  - Kernels:
  - Program objects:
  - Memory objects:
* Command queue
  - is created by a host and attached to a single OpenCL device
  - command type support
    - kernel execution commands
    - memory commands:
      - transfer data between the host and memory objects
      - move data between memory objects
      - map/unmap memory objects from the host address space
    - synchronization commands
  - commands are executed asynchronously
    - the host could continuous without waiting for commands to finish. If necessary, a explicit synchronization command is required.
  - commands within a single queue execute relative to each orther in one of two modes
    - in-order execution: commands are launched in order in which they appear in the command queue and complete in order
    - out-of-order(optional) execution: commands are issued in order but do not wait to complete before the following commands execute.
      - automatic load balancing: if a CU finishes its work early, it can immediately fetch a new command and start a new kernel.
  - An application is not done until all of kernels complete.
* Memory model
  - buffer object: contiguous block to store any data the programmer like
  - image object: hold images, including strage format
    - it's an opaque object manipulated by functions provided by OpenCL framework.
  - memory region:
    - host memmory: visible only to the host
    - global memory: read/write access to all work-items in all work-groups.
    - constant memroy: region of global memory remains constant during kernel execution.
      - Host allocates and initializes the memory objects.
      - Work-items have read-only to these objects.
    - local memory: local to work-group. allocates variables shared by all work-items in that work-group.
      - values in local memory are guaranteed to be consistent at work-group synchoronization points.
      - e.g. work group barrier requires all loads/stores defined before the barrier complete before any work-items in the group proceed past the barrier.
    - private memory: private to a work-item.
      - load/store memory into private memory canno be re-ordered to appear in any order
  - memory access:
    - copy/map/unmap as blocking or non-blocking.
* Programing model
  - data parallelism
    - no branch statements, each work-item will execute identical operations selected by its global ID. SIMD, Single Instruction Multiple Data.
    - with branch statements, each work-item uses the same "program". SPMD, Singlpe Program Multiple Data.
  - task parallelism, optional
    - many tasks are sent to out-of-order queue, it will be effective.

# How to build
* Mac
```
clang++ -g -framework OpenCL HelloWorld.cpp
```

# A general way of OCL app

* Query present platforms
* Query a set of devices supported by each platform:
  - Select devices, clGetDeviceInfo() on specific capabilities.
* Create contexts from a selected devices, each context must be created with devices from the a **single** platform.
  - create one or more command queues
  - create programs to run on one or more associated devices
  - Create a kernel from those programs
  - Allocate memory buffers and images, either on the host or on the devices.
  - Write or copy data to and from a particular devices
  - Submit kernels to a command queue for execution

# Platform

Get the list of platform
```
cl_int clGetPlatformIDs (cl_uint num_entries,
                         cl_platform_id *platforms,
                         cl_uint *num_platforms)
```
  * num_entries: the number of cl_platform_id entries that can be added to platforms
  * platforms: the array of platform
  * num_platforms: the number of platforms
e.g.
```
// get the number of platforms
errNum = clGetPlatformIDs(0, NULL, &numPlatforms);
platformIds = (cl_platform_id *)alloca(sizeof(cl_platform_id) * numPlatforms);

// get platformIds
errNum = clGetPlatformIDs(numPlatforms, platformIds, NULL);

// now we can iterate each platform
for (cl_unit i = 0; i < numPlatforms; i++) {
    Display(platformIds[i]);
}

// get the first platform directly
errNum = clGetPlatformIDs(1, &firstPlatformId, &numPlatforms);
```

Get the platform information
```
cl_int clGetPlatformInfo (cl_platform_id platform,
                          cl_platform_info param_name,
                          size_t param_value_size,
                          void *param_value,
                          size_t *param_value_size_ret)
```
  * the info may be different size, get size and get actual info
e.g.
```
// get platform name
err = clGetPlatformInfo(id, CL_PLATFORM_NAME, 0, NULL, &size);
char *name = (char *)alloca(sizeof(char) * size);
err = clGetPlatformInfo(id, CL_PLATFORM_NAME, size, name, NULL);

// get vendor name
err = clGetPlatformInfo(id, CL_PLATFORM_VENDOR, 0, NULL, &size);
char *vname = (char *)alloca(sizeof(char) * size);
err = clGetPlatformInfo(id, CL_PLATFORM_VENDOR, size, vname, NULL);
```

# Device
Given a platform, a list of supported devices can be queried with the command:
```
cl_int clGetDeviceIDs (cl_platform_id platform,
                       cl_device_type device_type,
                       cl_uint num_entries,
                       cl_device_id *devices,
                       cl_uint *num_devices)
```
  * each devicie share the same execution and memory model.
  * the "default" and "all" device option allow OCL runtime to assign a preferred device and all available devices respectively.
e.g.
```
cl_int errNum;
cl_uint numDevices;
cl_device_id deviceIds[1];

// get the number of devices
errNum = clGetDeviceIDs(platform, CL_DEVICE_TYPE_GPU, 0, NULL, &numDevices);

// get the first device
errNum = clGetDeviceIDs(platform, CL_DEVICE_TYPE_GPU, 1, &deviceIds[0], NULL);
```

Get device info
```
cl_int clGetDeviceInfo (cl_device_id device,
                        cl_device_info param_name,
                        size_t param_value_size,
                        void *param_value,
                        size_t *param_value_size_ret)
```
e.g. *all clGetXXInfo() will use the same way to get the size and info*
```
err = clGetDeviceInfo(deviceID, CL_DEVICE_MAX_COMPUTE_UNITS, sizeof(cl_unit), &maxComputeUnits, &size);
```
practice: App to get clinfo
  * CL_DEVICE_MAX_WORK_ITEM_SIZES:  array for each dimention
  * CL_DEVICE_MAX_WORK_GROUP_SIZE: the max number for a kernel to execute within a work group
```
      CL_DEVICE_MAX_WORK_ITEM_SIZES       CL_DEVICE_MAX_WORK_GROUP_SIZE
CPU:        1024    1       1                         1024
iGPU:       512     512     512                       512
dGPU:       1024    1024    1024                      256
```

# Context

Contexts provide a container for associated devices, memory objects and command queues(interface between ctx and dev).
  * OCL memory guarantees that all devices within the same context, will see these updates at well-defined synchronizatoin points. (memory in ctx could be synced among devices)
  * memory objects cannot be shared by different contexts(regardless of the same platform or not)
  * any data that is to be shared across contexts must be manually moved between contexts.

Create a context
```
cl_context clCreateContext (const cl_context_properties *properties,
                            cl_uint num_devices,
                            const cl_device_id *devices,
                            void (CL_CALLBACK *pfn_notify)(const char *errinfo,
                                                           const void *private_info,
                                                           size_t cb,
                                                           void *user_data),
                            void *user_data, cl_int *errcode_ret)


cl_context clCreateContextFromType (const cl_context_properties *properties,
                                    cl_device_type device_type,
                                    void (CL_CALLBACK *pfn_notify)(const char *errinfo,
                                                                   const void *private_info,
                                                                   size_t cb,
                                                                   void *user_data),
                                    void *user_data, cl_int *errcode_ret)
```
  * the `properties` are limited to below, other context properties are defined with certain OCL extensions.
```
    | cl_context_properties | Property value | Description                 |
    | CL_CONTEXT_PLATFORM   | cl_platform_id | specify the platform to use |
```
  * `pfn_notify` and `user_data` are used together to defince a callback to report info on errors.

e.g.
```
clGetDeviceInfo(platform, CL_DEVICE_TYPE_GPU, 0, NULL, &num);
if (num > 0) {
    devices = (cl_device_id *) alloca(num);
    // get all GPU devices
    clGetPlatformIDs(
        platform,
        CL_DEVICE_TYPE_GPU,
        num,
        &devices[0],
        NULL);
}

cl_context_properties properties[] = {
    CL_CONTEXT_PLATFORM, (cl_context_properties) platform, 0
};

// create ctx for all GPU devices
context = clCreateContext(
    properties,
    size / sizeof(cl_device_id),
    devices,
    NULL,
    NULL,
    NULL);
```

Get context info
```
cl_int clGetContextInfo (cl_context context,
                         cl_context_info param_name,
                         size_t param_value_size,
                         void *param_value,
                         size_t *param_value_size_ret)
```

Context reference count: like all OCL objects, it's also ref-counted.
```
cl_int clRetainContext(cl_context context)
cl_int clReleaseContext(cl_context context)
```

# Program with OpenCL C

Data Type
  * scalar: float
  * vector: float4
```
float4 c;
c.xyzw = (float4)(1.0f, 2.0f, 3.0f, 4.0f);
```
  * others:
    - image2d_t
    - image3d_t
    - sampler_t
    - event_t
  * derived type
    - the struct type cannot contain any pointers, if the struct or pointer to a struct is used as kernel argument.

Address Space Qualifiers
  * global: allocated from global memroy region
    - allow read/write access to all work-items in all work-groups executing a kernel.
    - global should *not* be used for image types
    - inside a generic function variables cannot be allocated in global address space, but global pointer is fine.
  * constant: allocated from global memory as read-only
    - image type cannot be allocated in the constant address space. (constant image2d_t imgA)
    - in program variables can be allocated only in constant address space rather than global
    - can be arguments for kernel, inside kernel as well with initialized value in compiling time.
  * local: allocated from local memory sharing by all work-items of a work-group, but not across work-groups
    - can be arguments for kernal and inside kernel *without* initialized value
    - must be allocated at kernel function scope(not in if block inside a kernel even)
    - lifetime is same asa work-group executing the kernel
  * private
    - by default, the generic address space for arguments to functions in a program, or local variables in a function.
    - private to a work-item, not shared between work-items or across work-groups.
  * casting
    - casting a pointer to another pointer in the *same* address space
```
kernel void my_func(global float4 *q)
{
    global float *p = (global float *)q;
    float *priv = (float *)q; // illegal
}
```

Access Qualifier
  * read_only(or \_\_read_only)
  * write_only(or \_\_write_only)

# OpenCL C Build-In Functions
Terms:
```
                             get_work_dim = 1
                          get_global_size = 16
   |<-------------------------------------------------------------------->|
   | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 |
   +---+---+---+---+---+---+---+---+---+----+----+----+----+----+----+----+
   | 6 | 3 | 4 | 9 | 1 | 1 | 2 | 2 | 4 | 5  | 8  | 3  | 4  | 2  | 5  | 7  |
   |<----------------------------->|<------------------------------------>|
          get_group_id = 0                    | get_local_size = 8
                                              |
                                              V
                                     get_local_id = 3
                                    get_global_id = 11
```
Execute the kernel on the command queue:
```
cl_int clEnqueueNDRangeKernel (cl_command_queue command_queue,
                               cl_kernel kernel,
                               cl_uint work_dim,                    // generally 1~3
                               const size_t *global_work_offset,    // start offset(0, ...), default is 0
                               const size_t *global_work_size,      // the total size of all data
                               const size_t *local_work_size,
                               cl_uint num_events_in_wait_list,
                               const cl_event *event_wait_list,
                               cl_event *event)
```
  * global_work_offset: start offet for global work items, default is 0.
  * global_work_size: the total size in each dimentions for caculation, regardless of offset, absolute number of global work items.
    - the total number of work-items in a work-group is computed as all dimentions of global_work_size[work_dim]
  * local_work_size: the number of work-items that make up a work-group, aka, the size of work-group
    - the total number of work-items in a work-group is computed as all dimentions of local_work_size[work_dim]
    - the total number(all dim) <= CL_DEVICE_MAX_WORK_GROUP_SIZE
    - for each dimention, local_work_size[work_dim] <= CL_DEVICE_MAX_WORK_ITEM_SIZES[work_dim]
    - if it's NULL, OCL will decide how to break the global work-items
    - it it's out of range(MAX size)
```
Error queuing kernel for execution.
```
e.g.
```

kernel:

__kernel void hello_kernel(__global const float *a,
                           __global float *result)
{
    int gid = get_global_id(0);

    result[gid] = a[gid];// + b[gid];
}

#define ARRAY_SIZE  1024

for (int i = 0; i < ARRAY_SIZE; i++) {
    a[i] = (float)i;
}

errNum = clSetKernelArg(kernel, 0, sizeof(cl_mem), &memObjects[0]);

size_t globalWorkOffset[1] = { 0 };
size_t globalWorkSize[1] = { 1024 };
size_t localWorkSize[1] = { 1 };

errNum = clEnqueueNDRangeKernel(commandQueue, kernel, 1,
                                globalWorkOffset,
                                globalWorkSize,
                                localWorkSize,
                                0, NULL, NULL);

CL_DEVICE_MAX_WORK_ITEM_SIZES:  512 512 512
CL_DEVICE_MAX_WORK_GROUP_SIZE:  512
```
  * print all with result memory
```
0 1 2 3 4 5 6 ... 1023
```
  * size_t globalWorkOffset[1] = { 3 };
```
0 0 0 3 4 5 6 7 8 9 10 11 12 13 ... 1023
```
  * size_t globalWorkSize[1] = { 256 };
```
0 1 2 3 4 5 6 ... 255 0 0 ... 0(1023nd)
```
  * globalWorkOffset[1] = { 3 } && globalWorkSize[1] = { 256 };
```
0 0 0 3 4 5 6 7 8 9 10 11 12 13 ... 255 256 257 258
```
  * globalWorkSize[1] = { 513 };
```
Error queuing kernel for execution.
```
  * ARRAY_SIZE = 1000, even if globalWorkSize > ARRAY_SIZE
```
0 1 2 3 4 5 6 ... 999
```

# Programs and Kernels
* kernel(\_\_kernel): the functions that will execute in parallel on a device
* kernel object(cl_kernel): set kernel arguments and query kernel info, created from kernel function
* program: includes many kernel functions, building into a program object
* program object(cl_program): can be created by either source code or a program binary
  - a container that stores the compiled executable code for each kernel on each device attached to it.

## Program
  * Create a program from external source code, usually *.cl file
```
cl_program clCreateProgramWithSource (cl_context context,       // inside a specific context
                                      cl_uint count,            // line number of source code
                                      const char **strings,     // all strings in a fulll source code
                                      const size_t *lengths,    // the length of each line, null-terminated
                                      cl_int *errcode_ret)
```
    - strings: from which the program object will be created.
  * Build program
```
cl_int clBuildProgram (cl_program program,
                       cl_uint num_devices,                     // number of device listed in device_list
                       const cl_device_id *device_list,         // device list for a program, NULL means for all devices
                       const char *options,                     // null-terminated string for building options
                       void (CL_CALLBACK *pfn_notify)(cl_program program,
                                                      void *user_data),
                       void *user_data)
```
    - return CL_SUCCESS if successful building
    - pfn_notify:
      - NULL, block to wait for building completion.
      - non-NULL, return directly, call pfn_notify when building is complete.
  * Get build info, if build error, get build log by CL_PROGRAM_BUILD_LOG
```
cl_int clGetProgramBuildInfo (cl_program program,
                              cl_device_id device,              // specify the device for which building info is required
                                                                // it must be one of devices for which the program was built
                              cl_program_build_info param_name,
                              size_t param_value_size,
                              void *param_value,
                              size_t *param_value_size_ret)
```
    - others: CL_PROGRAM_BUILD_STATUS, CL_PROGRAM_BUILD_OPTIONS, CL_PROGRAM_BUILD_LOG
  * Program build options
    - preprocessor option
      - `-D name`
      - `-D name=value`
      - `-I dir`
      - *Note*: the kernel fuction signatures(arguments list) must be the same for each device for which a single program object is built.
    - floating-point option
      - `cl-single-precision-constant`: if a constant is defined as double, treat it as single.
        - if (a <= 0.0) // 0.0 will be treated as single
      - `-cl-denorms-are-zero`: single and double numbers can be flushed to zero, except images.
    - optimizatin option
      - `-cl-opt-disable`: disable all optimizations, may by useful for debug
      - `-cl-strict-aliasing`: enable strict aliasing, careful that it may break the code. Below are different pointers.
        - short *ptr1
        - int *ptr2
    - Miscellaneous option
      - `-w`: disable the display of warning messages.
      - `Werror`: treat warnings as errors.
      - `-cl-std=version`: set the version of OCL C for compiler, e.g. CL1.1
  * Get program info:
```
cl_int clGetProgramInfo (cl_program program,
                         cl_program_info param_name,
                         size_t param_value_size,
                         void *param_value,
                         size_t *param_value_size_ret)
```
  * Create program from binary
```
cl_program clCreateProgramWithBinary (cl_context context,
                                      cl_uint num_devices,
                                      const cl_device_id *device_list,
                                      const size_t *lengths,
                                      const unsigned char **binaries,
                                      cl_int *binary_status,
                                      cl_int *errcode_ret)
```
  - we may create some helper func to save program binary and create program from binary
  - Note:
    - a program binary is valid only for the device with which it was created.
    - not safe to assume that a binary will work across other devices unless vendor specially gives this gurantee.
    - a program binary may or may not contain executable code. If it's an intermediate representation, OCL still need to compile it into the final executable.
  * update reference count(init as 1)
    - cl_int clReleaseProgram (cl_program program)
    - cl_int clRetainProgram (cl_program program)
  * it can finish compiler to unload any resouces consumed by the compiler.
    - cl_int clUnloadCompiler(void)

## kernel

* Create kernel object
```
cl_kernel clCreateKernel (cl_program program,
                          const char *kernel_name,  // the exact function name following __kernel keyword
                          cl_int *errcode_ret)      // non-NULL is error
```
* set kernel arguments
```
cl_int clSetKernelArg (cl_kernel kernel,
                       cl_uint arg_index,       // the index of kernel func, starting from 0
                       size_t arg_size,         // size of argument, which is determined by how the argument is declared in kernel function
                       const void *arg_value)
```
  - arg_size
    - local qualified: *number of bytes*, e.g. sizeof(cl_int) * NUM_OF_ARRAY
    - object: size of object type, e.g. sizeof(cl_mem)
    - sampler: sizeof(cl_sampler)
    - regular type: size of argument type. e.g. sizeof(cl_int)
  - arg_value
    - local qualified: *must be NULL*
    - object: a pointer to the memory object
    - sampler: a pointer to the sampler object
    - regular type: a ponter to the argument value
  - Note:
    - when it's called, the argument value will be internally copied by OCL, so it's safe to reuse the ponter for others.
  - e.g.
```
__kernel void func (global int *vertexArray,
                    int vertexCount,
                    float weight,
                    local float *localArray)
{ ... }

kernel = clCreateKernel(program, "func", NULL);

cl_int vertexCount;
cl_float weight;
cl_mem vertexArray;
cl_int localWorksize[1] = { 32 };

errNum = clSetKernelArg(kernel, 0, sizeof(cl_mem), &vertexArray);
errNum |= clSetKernelArg(kernel, 1, sizeof(cl_int), &vertexCount);
errNum |= clSetKernelArg(kernel, 2, sizeof(cl_float), &weight);
errNum |= clSetKernelArg(kernel, 3, sizeof(cl_float) * localWorkSize[0], &weight);

index 0: a point to cl_mem is a global pointer
index 3: local argument is only available inside a kernel function, value must be NULL
         type of localArray is cl_float, and the number of that is 32.
```
* create objects for all kernel functions in a program
```
cl_int clCreateKernelsInProgram (cl_program program,
                                 cl_uint num_kernels,   // NULL, to get the number of kernels in the program
                                 cl_kernel *kernels,
                                 cl_uint *num_kernels_ret)
```
* get kernel info
```
clGetKernelInfo (cl_kernel kernel,
                 cl_kernel_info param_name,
                 size_t param_value_size,
                 void *param_value,
                 size_t *param_value_size_ret)
```
* get kernel workgroup info
```
cl_int clGetKernelWorkGroupInfo (cl_kernel kernel,
                                 cl_device_id device,
                                 cl_kernel_work_group_info param_name,
                                 size_t param_value_size,
                                 void *param_value,
                                 size_t *param_value_size_ret)
```
* release kernel: a program object cannot be rebuilt until all of kernel objects associated with it have been released.
  - cl_int clReleaseKernel(cl_kernel kernel)

## Thread safty

All OCL API except clSetKernelArg() are thread thread-safe, that an App can have multiple threads simultineously call the same function without having to provide mutual exclusion.

clSetKernelArg()
  * the most frequently called function, to define it non-thread-safe to make it as fast as possible
  * no reason to set arguments for the same function in differetn theads
  * it could be used in multiple threads, but for different kernel functions

# Buffers and Sub-Buffers

* Memory object read/write can be marked as blocking util the enqueued command has completed
* Memory written to a particular device is visible to all devices in the same context
  - It's not possible to access other memory objects in different context.
  - Because a context is created for a particular platform, memory objects cannot be shared across different platform devices.
  - Solution:
    - copy data from dev1 in ctx1 to host
    - copy data from host to dev2 in ctx2
* Non-blocking thread will return before the enqueued command has completed, then it has to use below synchronization the ensure the command has completed:
  - cl_int `clFinish`(cl_command_queue queue): block until all pending commmands in the queue have completed.
  - cl_int `clWaitForEvents`(cl_unit num_events, const cl_event *event_list): block until all commands associated with corresponding events in events_list have completed.
* Memory objects only can

## Buffer creation

Create buffer
```
cl_mem clCreateBuffer (cl_context context,
                       cl_mem_flags flags,
                       size_t size,         // in bytes
                       void *host_ptr,      // pointer to data alocated by App
                       cl_int *errcode_ret)
```
  * flags: specify allocatoins and usage information
    - CL_MEM_READ_WRITE( *default* ): the allocation memory could be read and write
    - CL_MEM_READ_ONLY
    - CL_MEM_WRITE_ONLY
    - CL_MEM_USE_HOST_PTR: GTT BO. use the memory referenced by `host_ptr` as memory object, only if host_ptr is not NULL.
    - CL_MEM_ALLOC_HOST_PTR: visible VRAM BO. the buffer should be allcated from host-accessible memory.
      - cannot work with CL_MEM_USE_HOST_PTR, not valid.
    - CL_MEM_COPY_HOST_PTR: copy from `host_ptr` to device, GTT to (any)VRAM, only if host_ptr is not NULL.
      - cannot work with CL_MEM_USE_HOST_PTR
      - With CL_MEM_ALLOC_HOST_PTR to intiailie memory object allocated as host-accessible memory with host_ptr
        - copy from GTT to visible VRAM
  * host_ptr: pointer to data alocated by App
    - the size of memory pointed should be >= `size` in bytes.
  * Note:
    - memory object are set to kernel in global address space.
```
__kernel void square(__global float *buffer0)
{}

cl_mem buffer;
errNum = clSetKernelArg(kernel, 0, sizeof(cl_mem0, &buffer)
```

Sub-buffers provides a view into a particular buffer.
  * divide a single buffer into chunks that can be worked on independently as a ponter to a sub-memory area.
  * No need to take care of offset for each sub-buffer, but the offset is still required to create sub-buffers.
  * that's purely a software abstraction

Create sub-buffer
```
cl_mem clCreateSubBuffer (cl_mem buffer,                            // the entire/target buffer
                          cl_mem_flags flags,                       // same as clCreateBuffer()
                          cl_buffer_create_type buffer_create_type, // for sub-buffer info
                          const void *buffer_create_info,           // for sub-buffer info
                          cl_int *errcode_ret)
```
  * buffer_create_type: CL_BUFFER_CREATE_TYPE_REGION
  * buffer_create_info: a pointer to following structure
```
typedef struct _cl_buffer_region {
    size_t origin;
    size_t size;
} cl_buffer_region;
```
    - (origin, size) defines offset and size of sub-buffer in the original `buffer`
```
cl_mem buffer = clCreateBuffer(
    context,
    CL_MEM_READ_WRITE,
    sizeof(int) * NUM_BUFFER_ELEMENTS * numDevices,
    NULL,
    &errNum);

// now for all devices other than the first create a sub-buffer
for (unsigned int i = 1; i < numDevices; i++)
{
    cl_buffer_region region =
    {
        NUM_BUFFER_ELEMENTS * i * sizeof(int),
        NUM_BUFFER_ELEMENTS * sizeof(int)
    };
    buffer = clCreateSubBuffer(
            buffers[0],
            CL_MEM_READ_WRITE,
            CL_BUFFER_CREATE_TYPE_REGION,
            &region,
            &errNum);
    checkErr(errNum, "clCreateSubBuffer");

    buffers.push_back(buffer);
}

// create kernels for each sub-buffer
for (unsigned int i = 0; i < numDevices; i++)
{
    ...
    cl_kernel kernel = clCreateKernel(
            program,
            "square",
            &errNum);
    checkErr(errNum, "clCreateKernel(square)");

    errNum = clSetKernelArg(kernel, 0, sizeof(cl_mem), (void *)&buffers[i]);
    kernels.push_back(kernel);
    ...
}
```

Memory objects reference count
  * cl_int clRetainMemObject(cl_mem buffer)
  * cl_int clReleaseMemObject(cl_mem buffer)

Querying Buffers and Sub-Buffers
```
cl_int clGetMemObjectInfo (cl_mem memobj,
                           cl_mem_info param_name,
                           size_t param_value_size,
                           void *param_value,
                           size_t *param_value_size_ret)
```
  * CL_MEM_TYPE: for buffers and sub-buffers, returns CL_MEM_OBJECT_BUFFER
  * CL_MEM_FLAGS: return `flags` during buffer creation
  * CL_MEM_SIZE: in bytes
  * CL_MEM_HOST_PTR: return `host_ptr`
  * CL_MEM_ASSOCIATED_MEMOBJECT: (sub-buffer only) return the buffer from which it was created. otherwise NULL for general buffer
  * CL_MEM_OFFSET: (sub-buffer only) return offset, otherwise 0 for general buffer

## Read, write and copy

Read buffer: buffer object -> host memory
```
cl_int clEnqueueReadBuffer (cl_command_queue command_queue,
                        +-- cl_mem buffer,
                        |   cl_bool blocking_read,      // CL_TRUE, otherwise wait for the command event
                        |   size_t offset,
                        |   size_t size,
                        +-> void *ptr,
                            cl_uint num_events_in_wait_list,    // dependent event: number
                            const cl_event *event_wait_list,    // dependent event: list
                            cl_event *event)                    // generate event
```
```
cl_int clEnqueueReadBufferRect (cl_command_queue command_queue,
                            +-- cl_mem buffer,
                            |   cl_bool blocking_read,
                            |   const size_t *buffer_origin,    // (x, y, z) offset in the memory being read
                            |   const size_t *host_origin,      // (x, y, z) offset in the memory of ptr
                            |   const size_t *region,           // (w, h, d) size in bytes of 2D or 3D rectangle
                            |   size_t buffer_row_pitch,        // bytes in row for the memory being read
                            |   size_t buffer_slice_pitch,      // bytes in slice for the memory being read
                            |   size_t host_row_pitch,          // bytes in row for host memory
                            |   size_t host_slice_pitch,        // bytes in slice for host memory
                            +-> void *ptr,                      // pointer to host memory
                                cl_uint num_events_in_wait_list,
                                const cl_event *event_wait_list,
                                cl_event *event)
```
  * region: (width, height, depth), for 2D (w, h, 1). aka region[0, 1, 2]
  * buffer_row_pitch: bytes in row.
    - if it's 0, bytes in row is computed as region[0]
    - this can decide how to treat/seprate the buffer to ideal rectangle in row
  * buffer_slice_pitch: bytes in slice
    - if it's 0, bytes in slice is computed as region[1] * buffer_row_pitch
    - this can decide how to treat/seprate the buffer to ideal rectangle in slice
  * the offset into a memory region associated with the buffer is caculated, so does host_origin
```
buffer_origin[2] * buffer_slice_pith +
buffer_origin[1] * buffer_row_pith +
buffer_origin[0]
```

Write buffer: host memory -> buffer object
```
cl_int clEnqueueWriteBuffer (cl_command_queue command_queue,
                         +-> cl_mem buffer,
                         |   cl_bool blocking_write,    // CL_TRUE, otherwise wait for the command event
                         |   size_t offset,
                         |   size_t size,
                         +-- const void *ptr,
                             cl_uint num_events_in_wait_list,   // dependent event: number
                             const cl_event *event_wait_list,   // dependent event: list
                             cl_event *event)                   // generate event
```
```
cl_int clEnqueueWriteBufferRect (cl_command_queue command_queue,
                             +-> cl_mem buffer,
                             |   cl_bool blocking_write,
                             |   const size_t *buffer_origin,
                             |   const size_t *host_origin,
                             |   const size_t *region,
                             |   size_t buffer_row_pitch,
                             |   size_t buffer_slice_pitch,
                             |   size_t host_row_pitch,
                             |   size_t host_slice_pitch,
                             +-- const void *ptr,
                                 cl_uint num_events_in_wait_list,
                                 const cl_event *event_wait_list,
                                 cl_event *event)
```

Copy buffer: cl_mem src -> dst
```
cl_int clEnqueueCopyBuffer (cl_command_queue command_queue,
                            cl_mem src_buffer,
                            cl_mem dst_buffer,
                            size_t src_offset,
                            size_t dst_offset,
                            size_t size,
                            cl_uint num_events_in_wait_list,
                            const cl_event *event_wait_list,
                            cl_event *event)
```

e.g. for reading rectangle buffer
```
#define NUM_BUFFER_ELEMENTS 16

cl_int hostBuffer[NUM_BUFFER_ELEMENTS] =
{
    0, 1, 2, 3, 4, 5, 6, 7,
    8, 9, 10, 11, 12, 13, 14, 15
};

int ptr[4] = {-1, -1, -1, -1};
size_t buffer_origin[3]     = { 1 * sizeof(int), 1, 0 };
size_t host_origin[3]       = { 0, 0, 0 };
size_t region[3]            = { 2 * sizeof(int), 2, 1 };

//cl_int clEnqueueReadBufferRect (cl_command_queue command_queue,
//                            +-- cl_mem buffer,
//                            |   cl_bool blocking_read,
//                            |   const size_t *buffer_origin,    // (x, y, z) offset in the memory being read
//                            |   const size_t *host_origin,      // (x, y, z) offset in the memory of ptr
//                            |   const size_t *region,           // (w, h, d) size in bytes of 2D or 3D rectangle
//                            |   size_t buffer_row_pitch,        // bytes in row for the memory being read
//                            |   size_t buffer_slice_pitch,      // bytes in slice for the memory being read
//                            |   size_t host_row_pitch,          // bytes in row for host memory
//                            |   size_t host_slice_pitch,        // bytes in slice for host memory
//                            +-> void *ptr,                      // pointer to host memory
//                                cl_uint num_events_in_wait_list,
//                                const cl_event *event_wait_list,
//                                cl_event *event)
errNum = clEnqueueReadBufferRect (
                commandQueue,
                buffer,
                CL_TRUE,
                buffer_origin,
                host_origin,
                region,
                ( NUM_BUFFER_ELEMENTS / 4 ) * sizeof(int),  // separate the memory as 4 X 4 array
                0,
                0,
                2 * sizeof(int),    // works for 0 as well.
                static_cast<void *>(ptr),
                0,
                NULL,
                NULL);

res:
    5 6
    9 10

Note-1:
 ( NUM_BUFFER_ELEMENTS / 4 ) * sizeof(int),  // buffer_row_pitch separates the memory as 4 X 4 array

 0 | 0   1   2   3
 1 | 4   5   6   7
 2 | 8   9   10  11
 3 | 12  13  14  15

 ( NUM_BUFFER_ELEMENTS / 2 ) * sizeof(int),  // buffer_row_pitch separates the memory as 2 X 8 array

 0 | 0   1   2   3  4   5   6   7
 1 | 8   9   10  11 12  13  14  15

Note-2:
  row is in bytes
  height is in index
e.g.
size_t buffer_origin[3]     = { 1 * sizeof(int), 1, 0 };    // offset, (1, 1, 0)
size_t host_origin[3]       = { 0, 0, 0 };
size_t region[3]            = { 2 * sizeof(int), 2, 1 };    // size (2, 2, 0)


int ptr[6] = {-1, -1, -1, -1, -1, -1};
size_t region[3]            = { 2 * sizeof(int), 3, 1 };

res:
    5 6
    9 10
    13 14
```


## Map

Map a region of buffer object to host.
```
void * clEnqueueMapBuffer (cl_command_queue command_queue,
                           cl_mem buffer,               // target memory object
                           cl_bool blocking_map,        // wait until the memory is mapped to the host
                           cl_map_flags map_flags,      // R/W
                           size_t offset,               // in bytes, into buffer object
                           size_t size,
                           cl_uint num_events_in_wait_list,
                           const cl_event *event_wait_list,
                           cl_event *event,
                           cl_int *errcode_ret)
```
  * blocking_map: return when mapping is done. otherwise, user has to check the event status.
  * map_flags: CL_MAP_WRITE, CL_MAP_READ
```
cl_int * mapPtr = (cl_int*) clEnqueueMapBuffer(
    queues[0],
    buffers[0],
    CL_TRUE,
    CL_MAP_WRITE,
    0,
    sizeof(cl_int) * NUM_BUFFER_ELEMENTS * numDevices,
    0,
    NULL,
    NULL,
    &errNum);
```

After read/write, unmap the buffer
```
cl_int clEnqueueUnmapMemObject (cl_command_queue command_queue,
                                cl_mem memobj,          // buffer object being mapped
                                void *mapped_ptr,       // returned mapped pointer
                                cl_uint num_events_in_wait_list,
                                const cl_event *event_wait_list,
                                cl_event *event)
```

# Events

A platform defines a context that contents one or more devices, for each device there is one or more command queues.
Commands in a command queue generates events, and other commands can wait on these events before they execute.

The state of an event can describe the status of command.
  * CL_QUEUED: enqueued in command queue
  * CL_SUBMITTED: enqueued command has been submitted to device with command queue
  * CL_RUNNING: device is executing the command
  * CL_COMPLETE: the command is completed.

How to create events
the most comman way is to enqueue command
```
cl_int clEnqueueNDRangeKernel (cl_command_queue command_queue, -------+
                               cl_kernel kernel,                      |
                               cl_uint work_dim,                      |
                               const size_t *global_work_offset,      |
                               const size_t *global_work_size,        +--> must be in the same context(cmdq and wait events)
                               const size_t *local_work_size,         |
                               cl_uint num_events_in_wait_list,       |
                               const cl_event *event_wait_list, ------+
                               cl_event *event)
```
  * the command will not run until the events in list becomme the status of CL_COMPLETE or error return.
```
err = clEnqueueNDRangeKernel(commands, kernel1, 1, NULL, &glabol, &local, 0, NULL, &k_events[0]);
err = clEnqueueNDRangeKernel(commands, kernel2, 2, NULL, &glabol, &local, 0, NULL, &k_events[1]);
err = clEnqueueNDRangeKernel(commands, kernel2, 2, NULL, &glabol, &local, 2, k_events, NULL); // event = NULL, no need to generate event for others.
```

How to ignore events for a command
  * set the number of events 0
  * set the pointer to the array of events NULL

## Barrier
* Wait for events in the list to complete
* If the event list is empty, wait for all commands prior to this point complete before any of following commands start.
* Returned event can be used to identify this barrier command later on.
* *This command blocks following command execution, that is, any following commands enqueued after it do not execute until it completes.*
* e.g. Update memory objects across commands must be complete so that subsequent commands see the new values.
```
OpenCL 1.2

cl_int clEnqueueBarrierWithWaitList (cl_command_queue command_queue,
                                     cl_uint num_events_in_wait_list,
                                     const cl_event *event_wait_list,
                                     cl_event *event)

OpenCL 1.1 (deprecated in 1.2)

cl_int clEnqueueBarrier (cl_command_queue command_queue)
```
  * return error:
    - CL_INVALID_COMMAND_QUEUE: command queue it not valid
    - CL_INVALID_EVENT_WAIT_LIST: event number and list are not match, e.g. num > 0 but list is NULL
    - CL_OUT_OF_RESOURCES: fail to allocate resouces in device
    - CL_OUT_OF_HOST_MEMORY: fail to allocate resouces in host

## Marker
* Wait for events in the list to complete
* If the event list is empty, wait for all commands prior to this point complete before any of following commands start.
* Returned event can be used to identify this barrier command later on.
```
OpenCL 1.2

cl_int clEnqueueMarkerWithWaitList (cl_command_queue command_queue,
                                    cl_uint num_events_in_wait_list,
                                    const cl_event *event_wait_list,
                                    cl_event *event)
OpenCL 1.1 (deprecated in 1.2)
cl_int clEnqueueMarker (cl_command_queue command_queue,
                        cl_event *event)
different from clEnqueueBarrier(), it could generate an event to wait on.
```

## Synchronization

Wait events in OpenCL 1.1 only, deprecated in 1.2
```
cl_int clEnqueueWaitForEvents (cl_command_queue command_queue,
                               cl_uint num_events_in_wait_list,
                               const cl_event *event_wait_list)
```

For OpenCL 1.1
* that doesn't work, since barrier command only to the queue within which it's placed.
* Barrier is used in the same command queue, not across command queue.
```
clEnqueueNDRangeKernel()                        clEnqueueNDRangeKernel()
clEnqueueWriteBuffer()                          clEnqueueWriteBuffer()
clEnqueueReadBuffer()                           clEnqueueReadBuffer()

clEnqueueBarrier()  <----------------------->   clEnqueueBarrier()

clEnqueueNDRangeKernel()                        clEnqueueNDRangeKernel()
clEnqueueReadBuffer()                           clEnqueueReadBuffer()

```
* the sync across command queues
```
clEnqueueNDRangeKernel()                        clEnqueueNDRangeKernel()
clEnqueueWriteBuffer()                          clEnqueueWriteBuffer()
clEnqueueReadBuffer()                           clEnqueueReadBuffer()

clEnqueueBarrier()        event from Marker
clEnqueueWaitForEvents   <------------------    clEnqueueMarker(event), like barrier with event

clEnqueueNDRangeKernel()                        clEnqueueNDRangeKernel()
clEnqueueReadBuffer()                           clEnqueueReadBuffer()
```

For OpenCL 1.2, wait for events
```
cl_int clWaitForEvents (cl_uint num_events, const cl_event *event_list)
```
  * Waits on the host thread for commands identified by event objects in event_list to complete.
  * A command is considered complete if its execution status is CL_COMPLETE or a negative value.
  * error
    - CL_INVALID_VALUE if num_events is zero or event_list is NULL.
    - CL_INVALID_CONTEXT if events specified in event_list do not belong to the same context.
    - CL_INVALID_EVENT if event objects specified in event_list are not valid event objects.
    - CL_EXEC_STATUS_ERROR_FOR_EVENTS_IN_WAIT_LIST if the execution status of any of the events in event_list is a negative integer value.
    - CL_OUT_OF_RESOURCES if there is a failure to allocate resources required by the OpenCL implementation on the device.
    - CL_OUT_OF_HOST_MEMORY if there is a failure to allocate resources required by the OpenCL implementation on the host.

## Event Object

Reference count
* increment
```
cl_int clRetainEvent (cl_event event)
```
  - CL_SUCCESS: success
  - CL_INVALID_EVENT: event is not a valid event object
  - CL_OUT_OF_RESOURCES: fail to allocate resouces in device
  - CL_OUT_OF_HOST_MEMORY: fail to allocate resouces in host
* decrement
```
cl_int clReleaseEvent (cl_event event)
```

Get event information
```
cl_int clGetEventInfo (cl_event event,
                       cl_event_info param_name,
                       size_t param_value_size,
                       void *param_value,
                       size_t *param_value_size_ret)
```
  * CL_EVENT_COMMAND_QUEUE: return command queue associated with event
  * CL_EVENT_CONTEXT: return context associated with event
  * CL_EVENT_COMMAND_TYPE: e.g. CL_COMMAND_NDRANGE_KERNEL
  * CL_EVENT_COMMAND_ EXECUTION_STATUS:
    - CL_QUEUED: enqueued in command queue
    - CL_SUBMITTED: submitted to device already by the host to the device from command queue
    - CL_RUNNING: the device is executing the command
    - CL_COMPLETE: the command is completed
    - CL_EVENT_REFERENCE_COUNT: event reference count

## User defined event

Create user defined event for a specific context to wait
```
cl_event clCreateUserEvent (cl_context context, cl_int *errcode_ret)
```

Set event status by host, different from event for commands in device
  * it can be called only once for update user event status
  * status
    - CL_COMPLETE
    - a -n to indicate an error, causing all enqueued commands that wait on this user event to be terminated.
```
cl_int clSetUserEventStatus (cl_event event, cl_int execution_status)
```

Wait on host thread for commands identified by events in the event_list to complete
```
cl_int clWaitForEvents (cl_uint num_events, const cl_event *event_list)
```

## Set Callback

Set event callback, when the command associated with the event is CL_COMPLETE, event is called
```
cl_int
clSetEventCallback (cl_event event,
                    cl_int command_exec_callback_type,      // current only CL_COMPLETE
                    void (CL_CALLBACK *pfn_event_notify)
                         (cl_event event,
                          cl_int event_command_exec_status,
                          void *user_data),
                    void *user_data)
```
  * if multiple event set for the same command, these events will be executed in any order
  * callback cannot call blocking functions, e.g. clFinish, clWaitForEvents

## Usage

**Note**:(?)
  * Even if the command associated with the event has completed, cannot ensure the memory object modification by this command can be seen by other commands in the same command queue.
  * ref: https://software.intel.com/en-us/articles/opencl-using-events

Event is to sync commands among command queues in the same context, working for different command queues as well
```
cl_event k_events[2];
cmdq = clCreateCommandQueue(context, device, CL_QUEUE_OUT_OF_ORDER_EXEC_MODE_ENABLE, NULL);

err = clEnqueueNDRangeKernel(cmdq, kernel1, 1, NULL, &global, &local, 0, NULL, &k_events[0]);
err = clEnqueueNDRangeKernel(cmdq, kernel2, 1, NULL, &global, &local, 0, NULL, &k_events[1]);
err = clEnqueueNDRangeKernel(cmdq, kernel3, 1, NULL, &global, &local, 2, k_events, NULL);
```

Why not barrier, but event?
  * Barrier blocks any prior commands enqueued until they are all completed.
  * Event provides fine grained control for an out of order queue
  * Event works between commands in different queues, which are in the same context
  * Event convey more information than barrier, just complete or not.

Sync in different contexts
```
cl_event k_events[2];

// 2 cmdq in different contexts
cmdq1 = clCreateCommandQueue(context1, device_id1, CL_QUEUE_OUT_OF_ORDER_EXEC_MODE_ENABLE, &err);
cmdq2 = clCreateCommandQueue(context2, device_id2, CL_QUEUE_OUT_OF_ORDER_EXEC_MODE_ENABLE, &err);

// create user event on context2, who waits on.
cl_event uevent = clCreateUserEvent(context2, &err);

// commands in context1 creating events
err = clEnqueueNDRangeKernel(cmdq1, kernel1, 1, NULL, &global, &local, 0, NULL, &k_events[1]);
err = clEnqueueNDRangeKernel(cmdq1, kernel2, 1, NULL, &global, &local, 0, NULL, &k_events[2]);

// commands in context2 waiting on user event
err = clEnqueueNDRangeKernel(cmdq2, kernel3, 1, NULL, &global, &local, 0, uevent, NULL);

// wait on host for k_events
err = clWaitForEvents(2, &k_events);
err = clSetUserEventStatus(uevent, CL_COMPLETE);
```

## Profiling

Enable profiling in flag on command queue: CL_QUEUE_PROFILING_ENABLE
```
cl_int clGetEventProfilingInfo (cl_event event,
                                cl_profiling_info param_name,
                                size_t param_value_size,
                                void *param_value,
                                size_t *param_value_size_ret)   // actual size in bytes copied to [[param_value]]
```
  * nano second
  * resolution of a timer is value of CL_DEVICE_PROFILING_TIMER_RESOLUTION
  * param_name
    - CL_PROFILING_COMMAND_QUEUED: in command queue
    - CL_PROFILING_COMMAND_SUBMIT: host -> device
    - CL_PROFILING_COMMAND_START: start on the device
    - CL_PROFILING_COMMAND_END: finish on the device

e.g.
```
cl_command_queue commands = clCreateCommandQueue(context, device_id, CL_QUEUE_PROFILING_ENABLE, &err);

cl_event prof_event;

err = clEnqueueNDRangeKernel(commands, kernel, nd, NULL, global, NULL, 0, NULL, prof_event);
clFinish(commands);
err = clWaitForEvents(1, &prof_event);

cl_ulong ev_start_time = (cl_ulong)0;
cl_ulong ev_end_time = (cl_ulong)0;

err = clGetEventProfilingInfo(prof_event, CL_PROFILING_COMMAND_QUEUED, sizeof(cl_ulong, &ev_start_time, &return_bytes)


also can set callback to get profiling info, set user event to start to profile
```

## Events In Kernel
```
event_t async_work_group_copy (
                                __local gentype *dst,
                                const __global gentype *src, size_t num_gentypes,
                                event_t event)

event_t async_work_group_strided_copy ( __local gentype *dst,
                                        const __global gentype *src,
                                        size_t num_gentypes,
                                        size_t src_stride,
                                        event_t event)

void wait_group_events (int num_events, event_t *event_list)
```

# Images and Samplers

Image data could be emuated by generic memory object, which will lost performance comparing to work as image object.

Loading an image from a file by **FreeImage** libarary, and create an image object from its contents.
* OpenCL 1.1
```
cl_mem clCreateImage2D (cl_context context,
                        cl_mem_flags flags,
                        const cl_image_format *image_format,
                        size_t image_width,
                        size_t image_height,
                        size_t image_row_pitch,
                        void *host_ptr,
                        cl_int *errcode_ret)

cl_mem clCreateImage3D (cl_context context,
                        cl_mem_flags flags, // same enumeration cl_mem_flags
                        const cl_image_format *image_format,
                        size_t image_width,
                        size_t image_height,
                        size_t image_depth,
                        size_t image_row_pitch,
                        size_t image_slice_pitch,
                        void *host_ptr,     // A pointer to the image buffer laid out linearly in mem- ory.
                        cl_int *errcode_ret)
```
* OpenCL 1.2, to create 1D, 2D, 3D image buffer or array
```
cl_mem clCreateImage (cl_context context,
                      cl_mem_flags flags,
                      const cl_image_format *image_format,
                      const cl_image_desc *image_desc,      // info for image
                      void *host_ptr,
                      cl_int *errcode_ret)

typedef struct _cl_image_desc {
    cl_mem_object_type  image_type,
    size_t              image_width;
    size_t              image_height;
    size_t              image_depth;
    size_t              image_array_size;
    size_t              image_row_pitch;
    size_t              image_slice_pitch;
    cl_uint             num_mip_levels;
    cl_uint             num_samples;
    cl_mem              buffer;
} cl_image_desc;
```

Get Image information
```
cl_int clGetImageInfo (cl_mem image,
                       cl_image_info param_name,
                       size_t param_value_size,
                       void *param_value,
                       size_t *param_value_size_ret)
```

Image format is stored as cl_image_format, how individual pixels of the image are laid out in memory.
```
typedef struct _cl_image_format {
    cl_channel_order    image_channel_order;        // e.g. CL_R
    cl_channel_type     image_channel_data_type;    // e.g. CL_SNORM_INT8
} cl_image_format;
```

Samplers are required when fetching from an image object in kernel. Sampler tell the image-reading function how to access the image:
* Coordinate mode:
  - normalized in range [0..1]
  - Non-normalized in range [0..image_dim - 1]
* Addressing mode: behavior when fetching from an image with corrdinates that are outside the range of the image boundaries.
  - CL_ADDRESS_CLAMP: Coordinates outside the range of the image will return the border color.
  - CL_ADDRESS_CLAMP_TO_EDGE: Coordinates will clamp to the edge of the image.
  - CL_ADDRESS_REPEAT: Coordinates outside the range of the image will repeat.
  - CL_ADDRESS_MIRRORED_REPEAT: Coordinates outside the range of the image will mirror and repeat.
* Filter mode:
  - nearest: the value will be read from the image at the location nearest to the coordinate.
  - linear: several values close to the coordinate will be averaged.

```
cl_sampler clCreateSampler (cl_context context,
                            cl_bool normalized_coords,
                            cl_addressing_mode addressing_mode,
                            cl_filter_mode filter_mode,
                            cl_int *errcode_ret)
```
