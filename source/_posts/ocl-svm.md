---
title: OpenCL SVM
date: 2018-12-13 22:35:26
categories:
  - Compute
tags:
  - OpenCL
---

Introduction of SVM

<!--more-->

# Introduction #

OpenCL 2.0 shared vitual memory(SVM) enables the host and device portions of an OpenCL application to seamlessly share pointers and complex pointer-containing data-structures.
* Share vitual address space
* Defines memory model consistency guarantees for SVM allocations
  - This enables the host and the kernel sides to interact with each other using atomics for synchronization, like two distinct cores in a CPU.

1. `Shared virtual addressspace` between the host and a kernel on a device allows sharing pointer-based data structures between the host and the device.
1. `Identifying an SVM buffer using a regular pointer` without having to create a separate cl_mem object via the clCreateBuffer function.
  - This helps to integrate OpenCL into a legacy C/C++ program and to easily manage OpenCL memory resources on the host.
1. `"Map-free" access` to SVM allocations on the host side simplifies OpenCL host programming by eliminating the necessity to use map/unmap commands.
1. `Fine-grained coherent access` to an SVM allocation from the host during accessing the same SVM allocation from the kernel on the device side in the same time.
  - This allows the host and the device kernel to concurrently make modifications to adjacent bytes of a single SVM allocation.
1. `Fine-grained synchronization`: Concurrent modification of the same bytes from the host and from the kernel on the device using atomics enables light-weight synchronization and memory consistency between the host and the device without enqueueing new commands in an OpenCL command queue.
1. `Implicit use of any SVM allocation`: Pointers in one SVM allocation can point to other SVM allocations.
  - Minimum level of SVM support requires that such indirectly referenced allocations should be bound to a kernel's execution context or need to be explicitly passed as kernel parameters.
  - One of the advanced SVM features allows not passing all such indirectly used SVM allocations to kernels and using any number of them implicitly.
1. `Sharing the entire host address space` provided by an operating system seamlessly, without creating an SVM buffer for it.

* Coarse-grained buffer
  - 1, 2.
* Fine-grained buffer
  - w/o atomics: 1-4
  - w/ atomics: 1-6
* Fine-grained system
  - w/o atomics: 1-4, 6,7
  - w/ atomics: 1-7

# Detect supported SVM type #
Query svm capabilities from `clGetDeviceInfo()`
```
    cl_device_svm_capabilities caps;

    cl_int err = clGetDeviceInfo(
            deviceID,
            CL_DEVICE_SVM_CAPABILITIES,
            sizeof(cl_device_svm_capabilities),
            &caps,
            0
            );
```

Return a bit-filed variable containing below:
* CL_DEVICE_SVM_COARSE_GRAIN for coarse-grained buffer SVM
* CL_DEVICE_SVM_FINE_GRAIN_BUFFER for fine-grained buffer SVM
* CL_DEVICE_SVM_FINE_GRAIN_SYSTEM for fine-grained system SVM
* CL_DEVICE_SVM_ATOMICS for atomics support

# Identifying an SVM Buffer Using a Regular Pointer #
Allocate SVM buffer in *a given context* on device and return a regular pointer by `clSVMAlloc()`.
```
    void* p = clSVMAlloc (
            context,            // an OpenCL context where this buffer is available
            flags,              // access mode for the kernel and other options
            size,               // amount of memory to allocate (in bytes)
            0                   // alignment in bytes (0 means default)
            );
```
* it can only be used in the specified context
* flags:
  - access mode required for kernel is similar as clCreateBuffer()
    - CL_MEM_READ_ONLY - read-only memory when used inside a kernel
    - CL_MEM_WRITE_ONLY - memory is written but not read by a kernel
    - CL_MEM_READ_WRITE - memory is read and written by a kernel
  - memory in advanced types
    - CL_MEM_SVM_FINE_GRAIN_BUFFER - creates an SVM allocation that works correctly with fine-grained memory accesses
    - CL_MEM_SVM_ATOMICS - enables using SVM atomic operations to control visibility of updates in this SVM allocation 

Set kernel arguments by `clSetKernelArgSVMPointer()`
```
clSetKernelArgSVMPointer(kernel, 0, p);

kernel void mykernel (global float* p)
{
        . . .
}
```

Free SVM buffer
```
    clSVMFree (
            context,   // an OpenCL context used in corresponding clSVMAlloc call
            p          // a pointer to allocated with clSVMAlloc memory
            );
```
If need to synchronize the deallocation with OCL commands, enqueued into command queue.
  * clEnqueueSVMFree(): same as clSVMFree() to free the memory, with addition that enqueued as OCl command.

For compatibility, create cl_mem object on top of SVM buffer by calling `clCreateBuffer()` with `CL_MEM_USE_HOST_PTR` and passing `the pointer` that was returned from the `clSVMAlloc()`.
  * both buffer and p can be used to access the underlying SVM allocation
```
    cl_mem buffer = clCreateBuffer (
            context,            // an OpenCL context where this buffer is available
            CL_MEM_READ_ONLY | CL_MEM_USE_HOST_PTR,   // access mode for the device and other options
            size,               // amount of memory to allocate (in bytes)
            p,                  // pointer returned by clSVMAlloc
            &err                // resulting error code
            );
```

# Shared Virtual Address Space #
After allocated SVM buffer, the pointer assigned in host can be dereferenced in kernel on the device.
  - Both host and device use the same virtual address for SVM buffer.
  - In OpenCL 1.2, no SVM support, the shared data can be referred by indices

shared physical memory, different from shared virtual memory, may be available on OCL prior to 2.0.
  - in this case, host and device use different virtual memory for access to the same physical memory

# Map-free Access #
mapping machanism is required except fine-grained SVM buffer.

* coarse-grained SVM buffer
```
    float* p = (float*)clSVMAlloc(…);

    clEnqueueSVMMap(…,
            CL_TRUE,  // block until map is done
            p, …);


    // Initialize SVM buffer
    p[i] = …;

    clEnqueueSVMUnmap(…, p, …);

    clEnqueueNDRange(…);

    clEnqueueSVMMap(…,
            CL_TRUE,  // block until map is done
            p, …);

    // Read the data produced by the kernel
    … = p[i];

    clEnqueueSVMUnmap(…, p, …);
```
* fine-grained SVM buffer
```
    float* p = (float*)clSVMAlloc(…);
    // Initialize SVM buffer
    p[i] = …;

    clEnqueueNDRange(…);

    clFinish(…);

    // Read the data produced by the kernel
    … = p[i];
```
* To create fine-grained SVM buffer, with map-free access.
```
    void* p = clSVMAlloc (
            context,            // an OpenCL context where this buffer is available
            CL_MEM_READ_WRITE | CL_MEM_SVM_FINE_GRAIN_BUFFER,
            size,               // amount of memory to allocate (in bytes)
            0                   // alignment in bytes (0 means default)
            );
```

# Fine-Grained Coherent Access #

fine-grained SVM provides the ability to access the same memory region from host and device simultaneously.
* (different memory bytes in the same memory region)
* host could enqueue a kernel that modify the a memory, without waiting its finishing, access the same memory in host at the same time.
* host
```
    float* p = (float*)clSVMAlloc(…);

    clEnqueueNDRange(…, mykernel, …);
    clFlush(…);

    // Do not wait and modify some data
    p[0] = 0;
    p[2] = 2;
    p[4] = 4;

    clFinish(…);
---------------------------(time1)
```
* kernel
```
    kernel void mykernel (global float* p)
    {

        p[1] = 1;
        p[3] = 3;
        p[5] = 5;

    }
---------------------------(time1)
```
* at time1, the host and device get the same view of memory at p, which value is {0.f, 1,f, 2.f, 3.f, 4.f, 5.f}

If the same bytes are accessed by host and device simultaneously, additional synchonization is required.
* atomics
* memory fence

After kernel is executed completely, the SVM memory will combine the modification from both host and kernel, even if those modificatoin are made in different bytes.

# Fine-Grained Synchronization #

fine-grained synchonization:
* atomics
  - safely update the same scalar variable from the host and devices
* memory consistency
  - ensure a read/write to a memory by one agent is visable to other agents and in correct order
```
For example, if a circular queue is implemented in an SVM allocation, then insertion of a new queue item and the update of the queue's next_item pointer variable made by, say, the host, must be seen by a device in the right order.
```

To use that synchonization, should allocate SVM as following. Then the SVM memory can hold variables that can be used in atomic operations.
```
    void* p = clSVMAlloc (
            context,            // an OpenCL context where this buffer is available
            CL_MEM_READ_WRITE | CL_MEM_SVM_FINE_GRAIN_BUFFER | CL_MEM_SVM_ATOMICS,
            size,               // amount of memory to allocate (in bytes)
            0                   // alignment in bytes (0 means default)
            );
```

# Sharing the Entire Host Address Space #

System SVM allows:
* kernel could access any data in host address space
* no need to call clSVMAlloc() for buffer allocation.
* any memory allocated in host by malloc() or new, could be acessed by kernel
* Even so, take care of data alignment required by OCL spec

Buffer SVM allocation:
```
    float* p = (float*)clSVMAlloc(
            context,
            CL_MEM_READ_WRITE |
            CL_MEM_SVM_FINE_GRAIN_BUFFER,
            size,
            0
            );

    clSetKernelArgSVMPointer(
            mykernel, p, …);

    clEnqueueNDRange(…, mykernel, …);
```
System SVM allocation:
```
    // _aligned_malloc is one of the
    // methods to allocate aligned
    // memory to ensure efficient data
    // processing in the kernel
    float* p = (float*)_aligned_malloc(
            size,
            sizeof(cl_float16)
            );

    clSetKernelArgSVMPointer(
            mykernel, p, …);

    clEnqueueNDRange(…, mykernel, …);
```

# Implicit Use of Any SVM Allocation #

Host should use one of below to pass kernel parameters:
* `clSetKernelArgSVMPointer()`: to pass a pointer to an SVM allocation as a kernel argument
* `clSetKernelExecInfo()`: to pass pointers to all SVM allocations that can be reached and accessed by a specific kernel, but are not passed as kernel arguments

fine-grained system SVM is that host is not required to explicitly allocate SVM buffer.
* Explicit indirect use, buffer SVM allocation
```
    struct Node {
        float value;
        Node* next;
    };

    Node* node1 = (Node*)clSVMAlloc(
            context,
            CL_MEM_READ_WRITE |
            CL_MEM_SVM_FINE_GRAIN_BUFFER,
            sizeof(Node),
            0
            );

    node1->value = 1.f;

    Node* node2 = (Node*)clSVMAlloc(
            context,
            CL_MEM_READ_WRITE |
            CL_MEM_SVM_FINE_GRAIN_BUFFER,
            sizeof(Node),
            0
            );

    node2->value = 2.f;

    // Link node1 to node2node1->next = node2;
    node2->next = 0;

    // Pass node1 as a kernel argument
    clSetKernelArgSVMPointer(
            mykernel, node1, …);

    // Pass node2; it will be indirectly
    // used by a kernel through node1->next
    clSetKernelExecInfo(
            mykernel,
            CL_KERNEL_EXEC_INFO_SVM_PTRS,
            sizeof(node2),
            &node2
            );

    clEnqueueNDRange(…, mykernel, …);

```
* Implicit indirect use, system SVM allocation
```
    struct Node {
        float value;
        Node* next;
    };

    Node* node1 = new Node;
    node1->value = 1.f;

    Node* node2 = new Node;
    node2->value = 2.f;

    // Link node1 to node2node1->next = node2;
    node2->next = 0;

    // Pass node1 as a kernel argument
    clSetKernelArgSVMPointer(
            mykernel, node1, …);

    // node2 is not passed explicitly,
    // however kernel can still access it
    // through node1->next
    clEnqueueNDRange(…, mykernel, …);
```

# Reference #
* [opencl 2.0 shared virtual memory overview](https://software.intel.com/en-us/articles/opencl-20-shared-virtual-memory-overview)
