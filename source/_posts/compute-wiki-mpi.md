---
title: MPI introduction
date: 2019-01-25 17:09:58
categories:
  - Compute
tags:
  - Wiki
---

Message Passing Interface (MPI) is a standardized and portable message-passing standard designed by a group of researchers from academia and industry to function on a wide variety of parallel computing architectures.

<!--more-->

# MPI Implementation

* OpenMPI
  - One of implementation of MPI
  - https://en.wikipedia.org/wiki/Open_MPI
* MPICH
  - MPICH is one of the most popular implementations of MPI
  - https://en.wikipedia.org/wiki/MPICH
* Diff of OpenMPI and MPICH
  > It is important to recognize how MPICH and Open-MPI are different, i.e. that they are designed to meet different needs. MPICH is supposed to be high-quality reference implementation of the latest MPI standard and the basis for derivative implementations to meet special purpose needs. Open-MPI targets the common case, both in terms of usage and network conduits.
  - https://stackoverflow.com/questions/2427399/mpich-vs-openmpi

* Others
  - MPICH 3.0.4 and later on Mac, Linux SMPs and SGI SMPs.
  - MVAPICH2 2.0a and later on Linux InfiniBand clusters.
  - CrayMPI 6.1.0 and later on Cray XC30.
  - SGI MPT 2.09 on SGI SMPs.
  - OpenMPI development version on Mac.

## ARMCI

ARMCI: Aggregate Remote Memory Copy Interface
The purpose of the Aggregate Remote Memory Copy (ARMCI) library is to provide a general-purpose, efficient, and widely portable remote memory access (RMA) operations (one-sided communication) optimized for contiguous and noncontiguous (strided, scatter/gather, I/O vector) data transfers.
ARMCI is compatible with MPI.
ARMCI is a standalone system that could be used to support user-level
libraries and applications that use MPI or PVM(Parrallel Virtual Machine).

```
MPI (Message Passing Interface) is a description of an interface that can be used to communicate between computational nodes to coordinate calculations. MPI alone is not a framework that can be used, you need an implementation of this Interface, which is available for a lot of systems and programming languages.
PVM is a complete system that can be used to distribute a calculation on multiple computers, from what I know about PVM it uses a messaging protocol, similar to MPI, internally. I think an advantage of PVM is that it handles a lot of overhead, that you would have to implement manually with MPI.
```
  - http://hpc.pnl.gov/armci/
### ARMCI-MPI
  - is completely rewritten implementation of ARMCI
  - https://wiki.mpich.org/armci-mpi/index.php/Main_Page

## UCX
* It's ROCm-aware
* http://www.openucx.org/introduction/

# MPI Communication

## Point-to-point communication
* `MPI_Send`
```
MPI_Send(
    void* data,
    int count,
    MPI_Datatype datatype,
    int destination,
    int tag,
    MPI_Comm communicator)

```
  - tag: indicate a specific message
  > Sometimes there are cases when A might have to send many different types of messages to B. Instead of B having to go through extra measures to differentiate all these messages, MPI allows senders and receivers to also specify message IDs with the message (known as tags). When process B only requests a message with a certain tag number, messages with different tags will be buffered by the network until B is ready for them.

* `MPI_Recv`
```
MPI_Recv(
    void* data,
    int count,
    MPI_Datatype datatype,
    int source,
    int tag,
    MPI_Comm communicator,
    MPI_Status* status)
```

> Even though the message is routed to B, process B still has to acknowledge that it wants to receive A’s data. Once it does this, the data has been transmitted. Process A is acknowledged that the data has been transmitted and may go back to work.

## Broadcast
* `MPI_Bcast`: one process(root) sends same data to all processes in a communicator.
  - One of the main uses of broadcasting is to send out user input to a parallel program, or send out configuration parameters to all processes.
```
        +-> 0 (A)
        |
        +-> 1 (A)
0(A) => |
        +-> 2 (A)
        |
        +-> 3 (A)

MPI_Bcast(
    void* data,
    int count,
    MPI_Datatype datatype
    int root,
    MPI_Comm communicator)
```

## Scatter
* `MPI_Scatter`: similar as `MPI_Bcast`, but sends chunks of an array to different processes **in the order of process rank**.
```
              +-> 0 (A)
              |
              +-> 1 (B)
0(A,B,C,D) => |
              +-> 2 (C)
              |
              +-> 3 (D)

MPI_Scatter(
    void* send_data,
    int send_count,
    MPI_Datatype send_datatype,
    void* recv_data,
    int recv_count,
    MPI_Datatype recv_datatype,
    int root,
    MPI_Comm communicator)
```

## Gather
* `MPI_Gather`: inverse of `MPI_Scatter`, gather data from all processes to single one process(root).
```
0 (A) --+
        |
1 (B) --+
        | => 0(A,B,C,D)
2 (C) --+
        |
3 (D) --+

MPI_Gather(
    void* send_data,
    int send_count,
    MPI_Datatype send_datatype,
    void* recv_data,
    int recv_count,
    MPI_Datatype recv_datatype,
    int root,
    MPI_Comm communicator)

```

## Allgather
* `MPI_Allgather`: like `MPI_Gather` + `MPI_Bcast`, similar as `MPI_Gather` without the root process.
```
0 (A) --+    +-- 0 (A,B,C,D)
        |    |
1 (B) --+    +-- 1 (A,B,C,D)
        | => |
2 (C) --+    +-- 2 (A,B,C,D)
        |    |
3 (D) --+    +-- 3 (A,B,C,D)

MPI_Allgather(
    void* send_data,
    int send_count,
    MPI_Datatype send_datatype,
    void* recv_data,
    int recv_count,
    MPI_Datatype recv_datatype,
    MPI_Comm communicator)
```

## Reduce
* `MPI_Reduce`: similar to `MPI_Gather` with the reduced result of all data.
```
0 (3) --+
        |
1 (2) --+ (MPI_SUM)
        | ========> 0(12)
2 (5) --+
        |
3 (2) --+

0 (3, 1) --+
           |
1 (2, 3) --+ (MPI_SUM)
           | ========> 0(12, 9)
2 (5, 4) --+
           |
3 (2, 1) --+

MPI_Reduce(
    void* send_data,
    void* recv_data,
    int count,
    MPI_Datatype datatype,
    MPI_Op op,
    int root,
    MPI_Comm communicator)
```

## Allreduce
* `MPI_Allreduce`: reduce the values and distribute the results to all processes.
```
0 (3, 1) --+           +-- 0 (12, 9)
           |           |
1 (2, 3) --+ (MPI_SUM) +-- 1 (12, 9)
           | ========> |
2 (5, 4) --+           +-- 2 (12, 9)
           |           |
3 (2, 1) --+           +-- 3 (12, 9)

MPI_Allreduce(
    void* send_data,
    void* recv_data,
    int count,
    MPI_Datatype datatype,
    MPI_Op op,
    MPI_Comm communicator)

```


# Ref

* https://en.wikipedia.org/wiki/Message_Passing_Interface
* http://pages.tacc.utexas.edu/~eijkhout/pcse/html/
* http://mpitutorial.com/
