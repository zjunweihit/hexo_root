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

# ARMCI

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
## ARMCI-MPI
  - is completely rewritten implementation of ARMCI
  - https://wiki.mpich.org/armci-mpi/index.php/Main_Page

# UCX
* It's ROCm-aware
* http://www.openucx.org/introduction/

# Ref

* https://en.wikipedia.org/wiki/Message_Passing_Interface
* http://pages.tacc.utexas.edu/~eijkhout/pcse/html/
* http://mpitutorial.com/
