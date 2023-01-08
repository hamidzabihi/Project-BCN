# Project-BCN
Electronic and Optical Properties of Boron-Carbon-Nitride (BCN) Monolayers calculations by CP2K  


Below you can find the details about how we use these inputs to performy the electron-hole cooling and recombination dynamics in BCN monolayers by performing NA-MD calculations in the extended Density functional theory (DFT) framework.

Here, we will use a 2D BCN Monolayer structure with 48 atoms.

The procedure adopted here is as follows:


    Geometry optimization
    TD-DFT calculations
    ENERGY calculations
    Hybrid functionals
    Molecular dynamics
    Molecular orbitals overlap
    Molecular orbitals time-overlap

We highly welcome improving the functionality of the input files in this repository. So, please feel free to share your inputs and timings with us if you used these inputs.

More details on how to run the code are brought in [Libra CompChemCyberTraining](https://github.com/compchem-cybertraining/Tutorials_Libra/tree/master/6_dynamics/2_nbra_workflows) repository. Detailed explanations about installation and running the CP2K inputs can be found in [here](https://github.com/compchem-cybertraining/Tutorials_CP2K).


Below you can find the details about how we use these inputs to perform NA-MD for electron-hole recombination dynamics in BCN monolayers with using DFT method.

## 1. Molecular dynamics

This folder contains CP2K inputs for molecular dynamics simulations.

## 2. Molecular orbitals overlaps

The detialed explanation about different inputs used in this folder for computing the molecular overlaps and time-overlaps using CP2K and Libra are brought in [here](https://github.com/hamidzabihi/Project-BCN/tree/main/5_overlaps).
This folder also contains all the VMD input files that can be used to plot multiple molecular orbitals on the same geometry using `together_mode` keyword.

## 3. Nonadiabatic couplings

The computation of the NACs requires the correct path to molecular orbitals overlaps and time-overlaps. Detailed explanation about the parameters used in the `
step3.py` files are brought in [thislink](https://github.com/hamidzabihi/Project-BCN/tree/main/6_nacs). The NACs are computed between excited states in mixed electron and hole excitation basis that are ordered based on their identities. 


## 4. Nonadiabatic molecular dynamics

The inputs in this folder are used to perform electron-hole recombination dynamics using NA-MD and are adapted to the most recent version of Libra v5.3.0. 
Below, we explain only about the parameters that you we need to change to perform electron-hole recombination dynamics, such as longer trajectories by repeating the Hamiltonian matrices, defined with `user_nsteps`. Other parameters are fully explained in details in [this link](https://github.com/hamidzabihi/Project-BCN/tree/main/7_namd).

`path_to_npz_files`: The full path to the folder where the NAC were computed. 

`istep`: The index of the initial step for reading the NAC files.

`fstep`: The index of the final step for reading the NAC files.

`igeo`: The initial geometry to start the dynamics from the read files from `istep` to `fstep`. For example, if you set `igeo = 10`, it  will start the dynamics from the `istep+10`th step. 

`power`: With this parameter, one can scale the NAC values by a factor of `sqrt(2)^power` which is equal to `2^(power/2)`. We do not scale the NAC in this study, using `power=0`.

`nfiles`: The number of files that were read. This value is `fstep-istep`.

`active_space`: The active space means that which states you want to involve in the dynamics. For example, to include only the ground state and the first excited state, you can set it to `range(0,2)`. You can include more states in the dynamics as desired. In this study, we choose an active space which includes all excited states with ~0.2 eV above the first excited state that differs for each monolayer size.

`user_nsteps`: This parameter is dependent on how long you want to do the dynamics. If this value is larger than `nfiles`, it will use a repeating Hamiltonian approach. So, you can set it to an arbitrary positive integer value. For example, you can run 10 ps dynamics using the 2 ps Hamiltonian files.

`isNBRA`: This keyword, in the `params_nbra`, is used only for FSSH, ID-A, and mSDM type of calculations. If it is set to 1, the Hamiltonian related properties are only computed for one of the trajectories. This is good for nanoscale structures and for dynamics that involve a high number of states and trajectories. For DISH scheme, this value should be set to `0`. Currently, large number of states with a large number of trajectories cannot be used for DISH. 


After setting up your input, you can submit the jobs through slurm for multiple geometries using `sh exe.sh`. If you do not have access to multiple nodes, you can uncomment `# for icond in range(igeo,fgeo):` line and set your geometries as desired (don't forget to uncomment `for icond in [igeo]:`!).

The results will be stored for each scaling factor power and initial geometry in `power_namd_res_igeo` folders for each initial state, scheme, and batch. 


We perform the electron-hole recombination dynamics in the spin adapted configuration basis (SAC) basis by multiplying the NAC values by `sqrt(2)` which can be done using:

```
hvib_im[:,0] *= np.sqrt(2)
hvib_im[0,:] *= np.sqrt(2)
```


## 5. Visualizing

The Python files in folder `8_plot_properties` are multiple files that are used for analyzing and plotting the properties of the generated data from previous steps. The user can modify these scripts as desired. One of the most important libraries that we use is `glob` which can be used to find files with specific names. For example, for finding the imaginary part of the Hamiltonians, one can use `glob.glob('/path/to/nac/folder/*im*')` and use the found files for plotting the average NAC maps. 

