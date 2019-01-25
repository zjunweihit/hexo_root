---
title: HPC Application Introduction
date: 2019-01-25 15:35:58
categories:
  - Compute
tags:
  - Wiki
---

Introduce HPC applications.

<!--more-->

# NWChem

Open Source High-Performance Computational Chemistry

运行在高性能并行超级计算机和通常工作站集群上的计算化学软件，可以用在大多数计算平台上。NWChem使用标准量子力学描述电子波函或密度，计算分子和周期性系统的特性，还可以进行经典分子动力学和自由能模拟。

NWChem software can handle:
* Biomolecules, nanostructures, and solid-state
* From quantum to classical, and all combinations
* Ground and excited-states
* Gaussian basis functions or plane-waves
* Scaling from one to thousands of processors
* Properties and relativistic effects

> Ref
  * [main page](http://www.nwchem-sw.org/index.php/Main_Page)
  * [baike](https://baike.baidu.com/item/NWChem/248346?fr=aladdin)

# GROMACS

GROMACS is a molecular dynamics application designed to simulate Newtonian equations of motion for systems with hundreds to millions of particles. GROMACS is designed to simulate biochemical molecules like proteins, lipids, and nucleic acids that have a lot of complicated bonded interactions.

GROMACS是用于研究生物分子体系的分子动力学程序包。它可以用分子动力学、随机动力学或者路径积分方法模拟溶液或晶体中的任意分子，进行分子能量的最小化，分析构象等。它的模拟程序包包含GROMACS力场(蛋白质、核苷酸、糖等)，研究的范围可以包括玻璃和液晶、到聚合物、晶体和生物分子溶液。GROMACS是一个功能强大的分子动力学的模拟软件，其在模拟大量分子系统的牛顿运动方面具有极大的优势。

> Ref
  * https://www.nvidia.com/en-us/data-center/gpu-accelerated-applications/gromacs/
  * https://baike.baidu.com/item/GROMACS/2243638?fr=aladdin
  * http://www.gromacs.org/

# RELION

RELION (REgularized LIkelihood OptimizatioN) implements an empirical Bayesian approach for analysis of electron cryo-microscopy (Cryo-EM 冷冻电镜). Specifically it provides methods of refinement of singular or multiple 3D reconstructions as well as 2D class averages. RELION is an important tool in the study of living cells.

基于贝叶斯理论的冷冻电镜三维图像处理

> Ref
  * https://ngc.nvidia.com/catalog/containers/hpc:relion

# Lattice QCD

Lattice(格子) QCD is a well-established non-perturbative(非微扰) approach to solving the quantum chromodynamics (QCD 量子色动力学) theory of quarks and gluons(胶子，理论上假设无质量的粒子). It is a lattice gauge(测量) theory formulated on a grid or lattice of points in space and time.

> Ref
  * https://en.wikipedia.org/wiki/Lattice_QCD
  * https://wenku.baidu.com/view/500ae080f71fb7360b4c2e3f5727a5e9846a2712.html

# NAMD

NAMD (NAnoscale Molecular Dynamics) is a production-quality molecular dynamics application designed for high-performance simulation of large biomolecular systems. Developed by University of Illinois at Urbana-Champaign (UIUC), NAMD is distributed free of charge with binaries and source code.

NAMD（NAnoscale Molecular Dynamics）是用于在大规模并行计算机上快速模拟大分子体系的并行分子动力学代码。NAMD用经验力场，如Amber，CHARMM和Dreiding，通过数值求解运动方程计算原子轨迹。

> Ref
  * http://www.ks.uiuc.edu/Research/namd/
  * https://baike.baidu.com/item/NAMD/1297990
  * https://www.nvidia.com/en-us/data-center/gpu-accelerated-applications/namd/

# HOOMD-Blue

HOOMD-blue is a general-purpose particle simulation toolkit. It scales from a single CPU core to thousands of GPUs.
You define particle initial conditions and interactions in a high-level python script. Then tell HOOMD-blue how you want to execute the job and it takes care of the rest.

> Ref
  * http://glotzerlab.engin.umich.edu/hoomd-blue/

# LAMMPS

LAMMPS is a classical molecular dynamics code with a focus on materials modeling. It's an acronym for Large-scale Atomic/Molecular Massively Parallel Simulator.

LAMMPS has potentials for solid-state materials (metals, semiconductors) and soft matter (biomolecules, polymers) and coarse-grained or mesoscopic systems. It can be used to model atoms or, more generically, as a parallel particle simulator at the atomic, meso, or continuum scale. 

LAMMPS即Large-scale Atomic/Molecular Massively Parallel Simulator，可以翻译为大规模原子分子并行模拟器，主要用于分子动力学相关的一些计算和模拟工作，一般来讲，分子动力学所涉及到的领域，LAMMPS代码也都涉及到了。
LAMMPS由美国Sandia国家实验室开发，以GPL license发布。可以支持包括气态，液态或者固态相形态下、各种系综下、百万级的原子分子体系，并提供支持多种势函数。且LAMMPS有良好的并行扩展性。

> Ref
  * https://lammps.sandia.gov/
  * https://baike.baidu.com/item/LAMMPS/153102

# OpenFOAM

OpenFOAM is the leading free, open source software for computational fluid dynamics (CFD), owned by the OpenFOAM Foundation and distributed exclusively under the General Public Licence (GPL). The GPL gives users the freedom to modify and redistribute the software and a guarantee of continued free use, within the terms of the licence.

OpenFOAM是一个完全由C++编写，在linux下运行，面向对象的计算流体力学（CFD）类库。
OpenFOAM跟商用的CFD软件Ansys Fluent，CFX类似，但其为开源的，采用类似于我们日常习惯的方法在软件中描述偏微分方程的有限体积离散化。2004年开始OpenFOAM一直作为免费使用的开源软件，目前有OpenFOAM和OpenFOAM-Extened两个版本，分别有Henry Weller负责的OpenCFD以及Hrvoje Jasak负责的Wikki公司分别维护。

> Ref
  * https://baike.baidu.com/item/OpenFOAM/3052023?fr=aladdin
  * https://openfoam.org/

# CP2K

CP2K is a freely available (GPL) program, written in Fortran 2003, to perform atomistic simulations of solid state, liquid, molecular and biological systems.

用于固态、液体、分子和生物体系的原子和分子模拟。方法包括从第一定律密度泛函方法，到参量化经典双体、多体势。CP2K并不执行传统的Car-Parrinello分子动力学。

使用的交换-相关泛含有：
  * 交换部分：Slater, VWN, Pade; Becke88, Perdew86, PBE
  * 相关部分：VWN, Pade; LYP, Perdew86, PBE
  * 其中自旋极化只用于Becke88。

CP2k包含Quickstep，使用高斯基和平面波混合基组，对大体系进行线性标度的密度泛函计算。
DKH1至DKH5全电子标量相对论方法。

> Ref
  * https://en.wikipedia.org/wiki/CP2K
  * http://www.quantumchemistry.net/node/62
  * https://www.cp2k.org/
