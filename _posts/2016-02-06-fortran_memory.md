---
layout: post
title:  "Scaling of memory allocation in Fortran 90"
subtitle: "Comparing the methods used for allocating memory in F90"
---

## Introduction

As part of a molecular dynamics code I have the following situation: 
- a subroutine that is called 10^6 - 10^9 times
- the subroutine requires many (~30) small (< 512 elements) arrays for scratch calculation

Currently, as this was legacy code from Fortran 77, the arrays are declared with the maximum size possible (512 elements).  I would like to investigate the efficiency(wrt to speed) of two different ways of allocating the scratch arrays:

**Static**: Allocating arrays with the maximum amount of memory (the F77 way), e.g.:


		program main
    		integer, parameter :: max_size = 512
    		...
		subroutine loop_work
    		real, dimension(max_size) :: array
    		...	


**Module**: Allocating the arrays dynamically before the main loop in a module and importing the module. 

		module dynamic_arrays
			...
			real, dimension(:), allocatable :: array
			...	
		program main
			...
			actual_size = 64
			call allocate(actual_size) 
			call loop_work()
			...
		subroutine allocate(size)
			...
			allocate(array(size))
		subroutine loop_work()
			use dynamic_arrays
			...

I would like to know which method is faster. If the static method is much faster then I would keep the code as-is.  If they are the approximately same or the module method is faster I would refactor the code to implement that method. The module method also has the benefit of making the code a lot easier to read. 

## Results

I investigated the scaling with respect to the size of the array being allocated (2 - 64 elements) and with respect to the number of arrays being allocated (1, 4 & 8). I generated the data using with some test code which can be found here: 

[Github repo](https://github.com/RobertArbon/fortran_memory_analysis "Github repo")

Error bars are plotted but are too small to be resolved. 

### Scaling with array size
![Scaling with array size](/assets/Array_size_scaling.png)
So both methods have the same scaling and are indistinguishable from each other as to the absolute timings. 

### Scaling with the number of arrays
![Scaling with number of arrays](/assets/Num_array_scaling.png)
The module method scales slightly better, although with a small number of arrays it performes slightly worse. 

### Conclusions

As the module method doesn't perform worse than simply statically declaring scratch arrays I will opt for that.  

As to why the two methods are so similiar I am unsure - some more reading on what exactly Fortran is doing when it declares variables is required.

Another method to investigate would be the use of pointers which  may speed up the code even further.  

