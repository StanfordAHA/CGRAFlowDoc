# Introduction
[![Document Online](https://img.shields.io/badge/document-online-green.svg)](https://stanfordaha.github.io/CGRAFlowDoc/)
[![Build Status](https://travis-ci.com/StanfordAHA/CGRAFlowDoc.svg?branch=master)](https://travis-ci.com/StanfordAHA/CGRAFlowDoc)

While advances in software tools and frameworks have enabled individuals to create interesting new applications/products in reasonable time frames, hardware designs take large teams multiple years. This disparity in required effort decreases hardware innovation and interest in the field.  To address this issue, we must make hardware/software systems easier and more fun to develop, which means that we need to enable a more “agile” hardware development flow – making it possible to easily (and quickly) modify an existing design and play with the resulting system. To foster this goal of agile hardware design, we propose to create an open source hardware/software tool chain to rapidly create and validate alternative hardware implementations and a new open-source system RISC-V/CGRA SoC which will enable rapid execution/emulation of the resulting design.

Today’s CAD tools are extremely powerful and allow us to create systems of enormous complexity.  Unfortunately, their evolutional development path and focus on achieving the best possible performance mean that both their run time and the ramp time to use the tools don’t fit into an agile design flow.  Our recent work on domain specific languages (DSLs) for creating hardware (Genesis2, FPGen, Darkroom, Rigel, Halide) has shown that a different approach is possible, and we base our new tool chain on this approach. In addition to extending this base work on generating hardware RTL, this research adds fast physical mapping of RTL to FPGAs and CGRAs, and automatic generation of hardware/software APIs to interface the new hardware into a RISC-V/Linux system.  In the spirit of agile design, we will be building this toolset in an agile manner, leveraging existing tools when possible. We will use image/vision computation as our initial domain, and we plan to work closely with Kayvon’s CMU/Intel center in application selection. This research involves two mutually dependent tasks: the design tool flow and the configurable SoC.  The design tool flow makes use of modern SMT solvers, and we expect to make contributions in both using SMT for these applications, and in the underlying solvers as well. Below we summarize the key challenges and approaches in each of these areas.