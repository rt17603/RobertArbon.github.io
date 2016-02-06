---
layout: post
title:  "Scaling of memory allocation in Fortran 90"
subtitle: "Comparing the methods used for allocating memory in F90"
date:
categories: 
---

## Introduction

As part of a molecular dynamics code I have the following situation: 
- a subroutine that is called 10^6 - 10^9 times
- the subroutine requires many (~30) small (< 512 elements) arrays for scratch calculation

Currently, as this was legacy code from Fortran 77, the arrays are declared with the maximum size possible (512 elements).  I would like to investigate the efficiency(wrt to speed) of two different ways of allocating the scratch arrays:

1. *Static*: Allocating arrays with the maximum amount of memory 
	integer, parameter :: max_size = 512
	...
	...
	real, dimension(max_size) :: array

2. *Module*: Allocating the arrays dynamically at the before the main loop in a module and importing the module. 
