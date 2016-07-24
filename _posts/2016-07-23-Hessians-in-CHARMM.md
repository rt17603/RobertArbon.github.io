---
layout: post
title:  "The CHARMM VIBRAN command and the Hessian"
category: molecular-dynamics
---

## Introduction
This post is about the [Vibran](https://www.charmm.org/charmm/documentation/by-version/c40b1/params/doc/vibran/) command in [CHARMM](https://www.charmm.org/charmm/) which is primarily for normal mode analysis but will also write out the first derivate vector and second derivative matrix, the [Hessian](https://en.wikipedia.org/wiki/Hessian_matrix), of the energy function. The output from this function is not labelled or documented adequately, so I provide here a brief discussion of the output and some example input and output files so you can investigate its properties yourself. 

## The setup

The test system is a single H2 molecule. It's orientied at 45 degress to the X, Y, and Z axis and stretched just beyond its equilibrium bond length so that none of the elements of the first and second derivate matrices are zero. As there are six degress of freedom (three for Hydrogen 1 and three for Hydrogen 2) we should expect that the first and second derivatives of this energy function should consist of:  

1. **First derivatives**: Six elements in total with the first three equal to the negative of the second three. 
2. **Second derivatives**: A 6 by 6 Hessian, with 36 elements, but as it is symmetric, only 21 of them will be unique. 

The input files I used are hosted [here](https://github.com/RobertArbon/charmm-vibran). 

CHARMM will calculate the elements of the Hessian in two ways: 

1. Using the analytic expressions coded into the energy and force routines. 
2. Using a finite difference method with just the energy and force routines. 


### Analytic method
Let's look at the analystic method first. Both the first and second derivatives are calculated by invoking the following command:


    open write unit 11 card name h2_hessian_analytic.out
    
    vibran
    write seco card unit 11
    end
    
    close unit 11



The output written to [`h2_hessian_analytic.out`](https://github.com/RobertArbon/charmm-vibran/blob/master/h2_hessian_analytic.out) isn't labelled, so I've copied the output and added comments (preceded with `!`). The elements of the Hessian I'm labelling as `H_X1_Y2` which means the derivative of the energy with respect to X1 and then Y2. If the elements of the Hessian are ordered X1, Y1, Z1 ..., then this element will be in row 1 and column 5, and also in row 5 and column 1 (as it is symmetric). 


    2   !Number of atoms
        5.3660349058    !Potential energy (kcal/mol)
      -49.1397881259      -49.1397881259      -69.4940664828 ! First derivative atom 1
       49.1397881259       49.1397881259       69.4940664828 ! First derivative atom 2
      311.7175224272    !H_X1_X1
      196.0944915428    !H_X1_Y1
      277.3191369339    !H_X1_Z1
     -311.7175224272    !H_X1_X2
     -196.0944915428    !H_X1_Y2
     -277.3191369339    !H_X1_Z2
      311.7175224272    !H_Y1_Y1
      277.3191369339    !H_Y1_Z1
     -196.0944915428    !H_Y1_X2
     -311.7175224272    !H_Y1_Y2
     -277.3191369339    !H_Y1_Z2
      507.8110169144    !H_Z1_Z1
     -277.3191369339    !H_Z1_X2
     -277.3191369339    !H_Z1_Y2
     -507.8110169144    !H_Z1_Z2
      311.7175224272    !H_X2_X2
      196.0944915428    !H_X2_Y2
      277.3191369339    !H_X2_Z2
      311.7175224272    !H_Y2_Y2
      277.3191369339    !H_Y2_Z2
      507.8110169144    !H_Z2_Z2
        0.0000000000        0.0000000000        0.0000000000  !  Coordinates atom 1
        0.4250000000        0.4250000000        0.6010400000  !  Coordinates atom 2


Note that there are 21 second derivate elements which is what we expect. 

### Finite difference method
The finite difference method is invoked by: 


    open write unit 10 card name h2_hessian_finite.out
    
    vibran
    write seco card fini tol 0.0001 step 0.005 unit 10
    end
    
    close unit 10 

The output from this command is rather terse: 


     THE FOLLOWING FIRST DERIVATIVE ELEMENTS DIFFER BY MORE THAN TOL=    0.000100
     A TOTAL OF     6 ELEMENTS WERE WITHIN THE TOLERANCE
     THE FOLLOWING SECOND DERIVATIVE ELEMENTS DIFFER BY MORE THAN TOL=    0.000100
     A TOTAL OF    21 ELEMENTS WERE WITHIN THE TOLERANCE
     THE TRACE OF THE HESSIAN FOR ALL SELECTED ATOMS IS     2262.492124


 What's going on?  The two keywords `tol` and `step` will tell us the answer. The finite difference routine uses the `step` value to calculate an estimate of the first and second derivatives. The answers are compared to the analytic values and then only those estimated values which are with `tol`x100% of the analytic value are reported. The above output is telling us that all of estimated values are within 0.01% of the analytic values.  

 So if we increase the step size then the estimates should get worse and more values will be reported.  Alternatively we can make the tolerance `0.0` and then all of the estimates will be reported. Let's increase the step size by 2 (the values of `tol` and `step` in the call above are the default values):

 
    THE FOLLOWING FIRST DERIVATIVE ELEMENTS DIFFER BY MORE THAN TOL=    0.000100
      1 X    -49.13978813    -49.15708983      0.01730170
      1 Y    -49.13978813    -49.15708983      0.01730170
      1 Z    -69.49406648    -69.51037968      0.01631319
      2 X     49.13978813     49.15708983     -0.01730170
      2 Y     49.13978813     49.15708983     -0.01730170
      2 Z     69.49406648     69.51037968     -0.01631319
    A TOTAL OF     0 ELEMENTS WERE WITHIN THE TOLERANCE
    THE FOLLOWING SECOND DERIVATIVE ELEMENTS DIFFER BY MORE THAN TOL=    0.000100
      1 X -  1 Y    196.09449154    196.07074374    196.07074374      0.02374780         2
      1 X -  2 Y   -196.09449154   -196.07074374   -196.07074374     -0.02374780         5
      1 Y -  2 X   -196.09449154   -196.07074374   -196.07074374     -0.02374780         9
      2 X -  2 Y    196.09449154    196.07074374    196.07074374      0.02374780        17
    A TOTAL OF    17 ELEMENTS WERE WITHIN THE TOLERANCE
    THE TRACE OF THE HESSIAN FOR ALL SELECTED ATOMS IS     2262.492124
 

This output shows that with a `step` size of `0.01` Angstroms, all estimates of the first derivatives and four estimates of the second derivatives differ from their analytic values by more that `0.01%`.   Note that the ordering of the second derivatives is the same as in the analytic case. 

 There are still a lot of other numbers here so I've cut this output down and added annotations:


     THE FOLLOWING FIRST DERIVATIVE ELEMENTS DIFFER BY MORE THAN TOL=    0.000100
    !  Atom#-Derivative   |Analytic        |Estimate          |Difference
       1 X                |-49.13978813    |-49.15708983      |0.01730170
    
     A TOTAL OF     0 ELEMENTS WERE WITHIN THE TOLERANCE
     THE FOLLOWING SECOND DERIVATIVE ELEMENTS DIFFER BY MORE THAN TOL=    0.000100
    !  Atom#-Derivatives    | Analytic       |Estimate 1      |Estimate 2        |Average   Difference   |Matrix element index
       1 X -  1 Y           |196.09449154    |196.07074374    |196.07074374      |0.    02374780         2
    
     A TOTAL OF    17 ELEMENTS WERE WITHIN THE TOLERANCE
     THE TRACE OF THE HESSIAN FOR ALL SELECTED ATOMS IS     2262.492124

So the output reports the atom number and derivative type, then the analytic values, then the finite difference estimates, the difference between analytic and estimated values and finally the matrix element index (in this case it runs 1 - 21). 
In the second derivatives case  `Estimate 1` is the estimate of `H_X1_Y1` and `Estimate 2` is the estimate of  `H_Y1_X1` (i.e. its transpose) and the difference is the average difference from the analytic value. 

Hope this helps! 
