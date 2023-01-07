# Project-BCN
Electronic and Optical Properties of Boron-Carbon-Nitride (BCN) Monolayers calculations by CP2K  

This repository contains all the input files used to study the electron-hole cooling and recombination dynamics in BCN monolayers by performing NA-MD calculations in the extended tight-binding (DFT) framework.

Below you can find the details about how we use these inputs to perform NA-MD for electron-hole recombination dynamics in BCN monolayers with using DFT method.

Here we use CP2K v7.1 which is compiled with GCC-8.3 compiler For TD-DFT with hybrid functionals. This is because in lower versions of CP2K the TD-DFT calculation does not converge for hybrid functional due to a problem with converging ADMM calculations. It is also worth noting that the computed results are done using Intel(R) Xeon(R) E5-2699 v4 @ 2.20GHz CPUs. For CP2K installation we recommend the use of ./install_too_chain.sh in the tools/toolchain folder of CP2K. For compilation with Intel parallel studio you can use the instructions given in XConfigure website.

Here, we will use a 2D BCN Monolayer as a test and which its cif file is available from the crystallography website. We will use the structure with 48 atoms.

The procedure adopted here is as follows:


    Geometry and cell optimization
    TD-DFT calculations
    Hybrid functionals
        Molecular dynamics
        Molecular orbitals overlap
        Molecular orbitals time-overlap

We highly welcome improving the functionality of the input files in this repository. So, please feel free to share your inputs and timings with us if you used these inputs.
