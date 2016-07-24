---
layout: post
title:  "The CHARMM Analysis command"
category: molecular-dynamics
---

## Introduction 
The [Analysis](https://www.charmm.org/charmm/documentation/by-version/c40b1/params/doc/analys/) command in CHARMM prints out the contributions to the potential energy for a selection of atoms.  The commands are somewhat confusing and the output even more so, so this post gives a quick explanation. 

## The setup

To demonstrate this feature I'm using the solvated DHFR enzyme system from standard CHARMM test suite.  The files are hosted [here](https://github.com/RobertArbon/charmm-analysis).

## Analysis command

The analysis command prints out the contributions to the total energy for a selection of atoms. To demonstrate this I'm going to print out the energy contributions for resiude 50 in the DHFR protein. 

There are two main types of call to the Analsysis command: 

1. `all` - this prints out only those energy terms which involve atoms in the selection. 
2. `any` - this prints out energy terms which have at least one atom in the selection. 

So the output from '`all`'' will be smaller than that from a call to '`any`'. 
The other option for the Analysis command is whether to print out non-bonded energy terms or not. I'm going for the non-default behaviour and ask CHARMM to print out the non-bonded energy terms.

Within the same input file I've included two calls to the Analysis command, one using the `all` option and one using the `any` option. These are invoked using: 

    open write unit 1 card name ./all_resid50.out

    anal term all nonb unit 1 sele resid 50 end 

    energy

    close unit 1

    open write unit 1 card name ./any_resid50.out

    anal term any nonb unit 1 sele resid 50 end 

    energy

    close unit 1


## Output

The output is similar in both cases. It is divided into 6 blocks punctuated by something like: 

    ANAL: BOND: Index        Atom-I                   Atom-J          
            Dist           Energy         Force            Parameters


 * The word after `ANAL` gives the type of energy term
 * `Index` refers to CHARMM's internal index of that interaction. 
 * `Atom-I/J` gives the full name of the atom, i.e. `[Atom Number] [segment ID] [residue ID] [Atom label]`
    * the `[Atom label]` is the label in the coordinate or PDB file, not the atom type. 
 * `Dist` - (or `Angle` or `Dihedral`) is the main argument to this particular term in the energy function. 
 * `Energy` and `Force` - self explanatory
 * `Parameters` are the parameters found in the corresponding parameter file. They are preceded by an integer which is an internal index used to represent that particular interaction. 

You can see the difference between the '`any`' and '`all`' functions by looking at the corresponding output files.  In [any_resid50.out](https://github.com/RobertArbon/charmm-analysis/blob/master/any_resid50.out) you can see energy terms between different residues: 

    ANAL: BOND: Index        Atom-I                   Atom-J          
            Dist           Energy         Force            Parameters
    ANAL: BOND>  794  787 5DFR 49   SER  C     789 5DFR 50   ILE  N   
           1.354447       0.033018       3.495256   84       1.345000     370.000000

This shows a bond stretch between the C atom of residue 49 (SER) and the N atom of residue 50 (ILE). 

Looking at [all_resid50.out](https://github.com/RobertArbon/charmm-analysis/blob/master/all_resid50.out) we can see first it is a lot smaller (mostly due to the lack of non-bonded interactions between residue 50 and other residues) and there are no interactions which don't involve residue 50. 

 Aside from the slighly akward formatting, the other confusing thing about the output is that the CHARMM energy function contains the following terms: 

 1. Bond stretches,
 2. Angle bends,
 3. Urey Bradley 1,3 interactions,
 4. Dihedrals,
 5. Improper dihedrals,
 6. Van Der Waals,
 7. Electrostatic terms,
 
 yet the label on the energy terms do not reflect this.   The labels used in the six blocks and what they actually represent are shown below:
 
 1. `BOND` - Bond stretch
 1. `ANGL` - Angle bends
 2. `BOND` - Urey Bradley
 1. `DIHE`  - Dihedral terms
 2. `DIHE` - Improper dihedrals
 1. `VDW` - Both Van Der Waals and Electrostatic interactions. 


Despite it's confusing output, I've found it quite useful in diagnosing problems in my work on creating EVB models of enzymes. 










