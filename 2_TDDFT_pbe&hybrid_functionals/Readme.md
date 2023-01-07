# Hybrid functionals with CP2K

CP2K can perform hybrid functional calculations with a relatively good speed. You need to install CP2K with `LBINT` and `LIBXC` packages to be able to perform hybrid functional 
calculations. To access a library of exchange-correlation functionals you can use [LIBXC](https://www.tddft.org/programs/libxc/functionals/previous/libxc-5.0.0/). For different versions of CP2K one
needs a specific version of `LIBXC` or `LIBINT`. So, it is recommended to use the `./install_tool_chain.sh` to compile CP2K or look for the correct version of these libraries and compile them manually and then use them in compilation of CP2K (for more information take a look over [this link](https://xconfigure.readthedocs.io/en/latest/cp2k/)). 
The key to do the hybrid functional calculation speed is the initial guess for the SCF cycle. 
In order to do so, we need to first obtain an initial converged PBE `wfn` file and then use the PBE `wfn` file for the initial SCF guess of the hybrid functional. Here, we have
provided three files: `pbe.inp`, `hse06.inp`, and `b3lyp.inp`. The `pbe.inp` file runs a pure functional calculations and after the complete convergence, produces a `wfn` file. We use this file as an initial 
guess for the `hse06.inp` and `b3lyp.inp` using the `SCF_GUESS RESTART` and `WFN_RESTART_FILE_NAME BA2_PbI4_PBE-RESTART.wfn`.
For HSE06 caclulations we have adopted an input from [here](https://www.cp2k.org/_media/events:2018_summer_school:cp2k-uk-stfc-june-2018-sanliang-ling.pdf). We can use the
PBE potentials for HSE06 but for B3LYP, we need to use the BLYP potentials. The BLYP potentials are available for different atoms in [`GTH_POTENTIALS`](https://github.com/mkrack/cp2k-data/blob/master/potentials/Goedecker/cp2k/GTH_POTENTIALS) file.

In hybrid functional calculations, we need to add this part for HSE06 for the `&XC` section (the `&VDW_POTENTIAL` part is added in the input):
```
&XC
  &XC_FUNCTIONAL
  &XWPBE
    SCALE_X -0.25
    SCALE_X0 1.0 
    OMEGA 0.11
  &END
  &PBE
    SCALE_X 0.0
    SCALE_C 1.0
  &END PBE
  &END XC_FUNCTIONAL
  &HF
    &SCREENING
      EPS_SCHWARZ 1.0E-6
      EPS_SCHWARZ_FORCES 1.0E-5
      SCREEN_ON_INITIAL_P FALSE
    &END SCREENING
    &INTERACTION_POTENTIAL
      CUTOFF_RADIUS 10
      POTENTIAL_TYPE SHORTRANGE
      OMEGA 0.11
      !T_C_G_DATA t_c_g.DAT
    &END INTERACTION_POTENTIAL
    ! Defines the maximum amount of memory [MiB] to be consumed by the full HFX module.
    !&MEMORY
    !  MAX_MEMORY  10000
    !  EPS_STORAGE_SCALING 0.1
    !&END MEMORY
    FRACTION 0.25
  &END HF
&END XC
```
For hybrid functionals, CP2K uses Auxiliary Density Matrix Method (ADMM). This needs to be added to the `&DFT` section:
```
&AUXILIARY_DENSITY_MATRIX_METHOD
  ! recommended, i.e. use a smaller basis for HFX
  ! each kind will need an AUX_FIT_BASIS_SET.
  METHOD BASIS_PROJECTION
  ! recommended, this method is stable and allows for MD. 
  ! can be expensive for large systems
  ADMM_PURIFICATION_METHOD MO_DIAG
&END
```
Note that the `MO_DIAG` in `ADMM_PURIFICATION_METHOD` is good and stable for MD but also note that this method works only for `OT` method. In order to do the TD-DFT calculations you need to set this to `NONE`.
In the `&DFT` section, this part should also be added. The files `BASIS_ADMM` or `BASIS_ADMM_MOLOPT` include the auxiliary basis set for each atom type.
There is no problem to add multiple `BASIS_SET_FILE_NAME` to the input and the CP2K will concatenate them but one should not add multiple keywords in CP2K like `POTENTIAL_FILE_NAME`.
```
BASIS_SET_FILE_NAME BASIS_MOLOPT
BASIS_SET_FILE_NAME BASIS_ADMM
BASIS_SET_FILE_NAME BASIS_ADMM_MOLOPT
POTENTIAL_FILE_NAME GTH_POTENTIALS
```
For each atom type, two basis set should be added for hybrid functional calculations. The first one is the usual basis set and the second one is the auxiliary basis set
and is defined with `AUX_FIT` after `BASIS_SET`. Here is an example for Pb atom:
```
&KIND Pb
  BASIS_SET DZVP-MOLOPT-SR-GTH 
  BASIS_SET AUX_FIT cFIT6
  POTENTIAL GTH-PBE-q4
&END KIND
```
For B3LYP, we need to use `LIBXC` [library functionals](https://www.tddft.org/programs/libxc/functionals/previous/libxc-5.0.0/) which is `XC_HYB_GGA_XC_B3LYP`. You can also use other B3LYP functionals such as `HYB_GGA_XC_CAM_B3LYP` which is CAM-B3LYP. Here is the `&XC` section for B3LYP (the `&VDW_POTENTIAL` part is added in the input):
```
&XC
  &XC_FUNCTIONAL
    &LIBXC
      FUNCTIONAL XC_HYB_GGA_XC_B3LYP
    &END LIBXC
  &END XC_FUNCTIONAL
  &HF
    &SCREENING
      EPS_SCHWARZ 1.0E-10
    &END
    !&MEMORY
    ! This is the maximum memory for each processor in MBi, I just comment it but
    ! you can obtain it through computing the memory you ask in the slurm, pbs, or bash file
    ! divided by the number of processors.
    !  MAX_MEMORY  10000
    !  EPS_STORAGE_SCALING 0.1
    !&END MEMORY
    FRACTION 0.20
  &END
&END XC

```
In the `&KIND` section we need to add `BLYP` potentials from `GTH_POTENTIALS`. Here is the example for Pb atom:
```
&KIND Pb
  BASIS_SET DZVP-MOLOPT-SR-GTH
  BASIS_SET AUX_FIT cFIT6
  POTENTIAL GTH-BLYP-q4
&END KIND
```
By setting the `&MO_CUBES` and printing out only the energies by `WRITE_CUBE .FALSE.`, you can see that the energy gaps have increased compared to PBE functional calculations.

In this table you can see the timings needed for running the hybrid functional calculations with and without using the PBE converged `wfn` file. Here we used 25 processors (CP2K v7.1). Although the timing is not so much different but this will show itself for larger systems or some other systems which PBE convergence is easier and the hybrid functional will be converged in a couple of steps. Overall, it can be used as an alternative for the convergence of hybrid functionals.

|  Functional | Elapsed time (s) | 
|---|---|
|PBE|  171.1    |
|HSE06 with PBE `wfn` file   |   387.5 (Initial step: 139.4, 16 steps with 15.5)   |
|B3LYP with PBE `wfn` file   |  359.3 (Initial step: 57, 16 steps with 18.9)       |
|HSE06 without PBE `wfn` file|   408.2 (Initial step: 138.1, 26 steps with 15.7)    |
|B3LYP without PBE `wfn` file|   532 (Initial step: 56.9, 28 steps with 19)   |

Note that for TD-DFT with hybrid functionals we need to use the CP2K v7 or higher. In lower versions, there is a known problem with convergence of the TD-DFT calculations
with ADMM which seems to be fixed in v7. The same as before it is needed to add this section for TD-DFT calculations:
```
&PROPERTIES
  &TDDFPT
     NSTATES     20            # number of excited states
     MAX_ITER    200           # maximum number of Davidson iterations
     CONVERGENCE [eV] 1.0e-5   # convergence on maximum energy change between iterations
     &MGRID
        NGRIDS 16
        CUTOFF 500 # separate cutoff for TDDFPT calc
     &END
     ! Only in case you have a tdwfn file from previous calculations
     !RESTART     .TRUE.
     !WFN_RESTART_FILE_NAME RESTART.tdwfn
  &END TDDFPT
&END PROPERTIES
```
We have also added this part to the input although it is not needed since from now on for TD-DFT calculations with hybrid functionals we use the CP2K v7.1.
```
&XC_GRID
  XC_DERIV SPLINE2_SMOOTH
&END XC_GRID
```
The timings for the TD-DFT calculations for the B3LYP are shown in the table below. Here we used 25 processors (CP2K v7.1).
| #Excited states | Time for each TD-DFT cycle (s) |Total elapsed time (s)| Maximum excitation energy (eV) |
|---|---|---|---|
|20   | 363  |  6183 | 0.82 |
| 40  |  720 |  8639 | 0.98  |

The results of the TD-DFT calculations with PBE functional was shown in the [tddft] section. The band gaps for different functionals are shown in the following table. Note that with hybrid functionals the states energy gaps are higher than the pure functional. The maximum excitation energy with 20 states is 0.82 eV while for PBE functional it was 0.48 eV. So, dependent on the experiments, we can see that we can adopt even lower number of states for this functional.

|Functional | Band gap (eV) |
|---|---|
|PBE| 2.416|
|HSE06|  3.193  |
|B3LYP|  3.522  |

Finally, we have provided the B3LYP excitation analysis below. For the B3LYP hybrid functional, the TD-DFT calculations shows more combination of states and the excitonics effects are higher than PBE functional for the 2D BCN monolayer. Also, less dark states can be observed compared to PBE functional. Another important point is that using B3LYP, the initial excitation energy has reduced. We can conclude that although the band gap has not a good agreement with experiments, in TDDFT for the first excited state, it will get closer to the experiments.
```

         State    Excitation        Transition dipole (a.u.)        Oscillator
         number   energy (eV)       x           y           z     strength (a.u.)
         ------------------------------------------------------------------------
 TDDFPT|      1       1.16123   1.9640E+00 -1.0388E-11  1.0077E+01   2.99848E+00
 TDDFPT|      2       1.25014   7.2288E+00 -1.4712E-11  2.3680E-01   1.60221E+00
 TDDFPT|      3       1.33182  -9.1907E+00 -3.0592E-12  3.8870E+00   3.24914E+00
 TDDFPT|      4       1.43693   1.9236E+00  3.0173E-13  7.9876E+00   2.37636E+00
 TDDFPT|      5       1.50449  -4.0797E-06 -3.8085E-15  1.1041E-05   5.10669E-12
 TDDFPT|      6       1.59056  -1.0862E-05 -1.1339E-15 -1.0577E-05   8.95749E-12
 TDDFPT|      7       1.67159  -5.2915E-07  1.8513E-15  5.2756E-06   1.15127E-12
 TDDFPT|      8       1.87247  -3.0086E-06 -3.4239E-15 -1.4358E-05   9.87248E-12
 TDDFPT|      9       1.92779   2.4921E-01  1.4719E-11  6.8799E+00   2.23844E+00
 TDDFPT|     10       2.03763  -2.8070E-06  2.7403E-15  1.2505E-06   4.71419E-13
 TDDFPT|     11       2.12232  -3.5593E-01 -7.2179E-12  1.0899E-01   7.20494E-03
 TDDFPT|     12       2.17545  -9.3020E-07  3.3340E-14  4.5328E-06   1.14119E-12
 TDDFPT|     13       2.24544   9.6197E-07  1.4581E-14  9.8696E-07   1.04495E-13
 TDDFPT|     14       2.27453  -2.4717E-01  7.0369E-12 -7.3167E-01   3.32361E-02
 TDDFPT|     15       2.33647   4.0787E-01 -4.8846E-12  1.0800E+00   7.62903E-02
 TDDFPT|     16       2.38349   9.1837E-08 -3.9913E-14 -7.8854E-06   3.63148E-12
 TDDFPT|     17       2.46109   5.9236E-01 -5.7509E-12 -3.5428E-01   2.87253E-02
 TDDFPT|     18       2.57865   1.2904E+00  1.0531E-11 -1.0568E-01   1.05894E-01
 TDDFPT|     19       2.61095  -7.8771E-06  8.3172E-15 -1.2205E-06   4.06433E-12
 TDDFPT|     20       2.69513   3.6347E-06 -7.8345E-15 -8.4184E-07   9.19132E-13

 TDDFPT : CheckSum  =  0.334987E+00
 
 -------------------------------------------------------------------------------
 -                            Excitation analysis                              -
 -------------------------------------------------------------------------------
        State             Occupied              Virtual             Excitation
        number             orbital              orbital             amplitude
 -------------------------------------------------------------------------------
             1   1.16123 eV
                                96                   97               0.994514
                                96                   98               0.062100
             2   1.25014 eV
                                96                   98              -0.973689
                                95                   97              -0.164881
                                94                   99              -0.066970
                                96                   97               0.062597
                                95                   98               0.061849
             3   1.33182 eV
                                95                   97              -0.977203
                                96                   98               0.163598
                                94                   99              -0.050715
             4   1.43693 eV
                                95                   98               0.986889
                                96                   98               0.058010
                                93                   99              -0.057755
             5   1.50449 eV
                                96                   99              -0.953359
                                94                   98              -0.232514
                                95                   99              -0.140964
                                96                  100               0.094574
             6   1.59056 eV
                                94                   97               0.878351
                                95                   99               0.412414
                                94                   98               0.187139
                                96                   99              -0.108665
                                96                  100              -0.070136
             7   1.67159 eV
                                94                   98              -0.761999
                                95                   99               0.602409
                                96                  100              -0.129703
                                94                   97              -0.114999
                                96                   99               0.083541
                                93                   97              -0.073158
                                93                   98              -0.058361
                                95                  100              -0.050285
             8   1.87247 eV
                                96                  100               0.602973
                                95                   99               0.507986
                                94                   98               0.354256
                                94                   97              -0.272177
                                95                  100               0.194146
                                93                   97              -0.182838
                                93                   98              -0.177716
                                96                  102               0.162345
                                94                  101               0.104375
                                95                  103               0.098228
                                96                   99              -0.096895
             9   1.92779 eV
                                94                   99              -0.981917
                                94                  100               0.084388
                                92                   98               0.080345
                                93                   99              -0.067163
                                95                  101              -0.066410
                                95                   97               0.054482
                                96                   98               0.050593
            10   2.03763 eV
                                95                  100               0.749150
                                96                  100              -0.561714
                                94                   98               0.152928
                                94                   97              -0.146564
                                95                   99               0.106434
                                92                   99              -0.104100
                                94                  101               0.098844
                                93                   98              -0.093922
                                96                   99              -0.092470
                                95                  102               0.069546
                                94                  104               0.067713
            11   2.12232 eV
                                96                  101              -0.899063
                                95                  101               0.364969
                                92                   98               0.171878
                                92                   97              -0.112285
                                94                  100              -0.078097
            12   2.17545 eV
                                95                  100              -0.516644
                                93                   97              -0.472745
                                96                  100              -0.440824
                                94                   98               0.272891
                                93                   98              -0.256501
                                94                  101              -0.167429
                                90                   97              -0.162691
                                94                   97              -0.162322
                                95                   99               0.136527
                                92                   99              -0.123237
                                96                   99              -0.115590
                                91                   99               0.091325
                                95                  102               0.074043
                                96                  103              -0.073168
                                96                  102               0.051725
                                93                  101               0.051487
                                96                  105               0.051096
            13   2.24544 eV
                                93                   97              -0.702091
                                93                   98               0.648006
                                95                  100               0.168681
                                92                   99               0.112754
                                96                  102              -0.077813
                                96                  100               0.066680
                                94                  101              -0.060727
                                96                   99               0.057650
                                94                   97               0.057300
                                95                   99              -0.054091
                                91                  100              -0.053285
                                93                  101              -0.050572
            14   2.27453 eV
                                92                   97               0.610126
                                95                  101              -0.546265
                                94                  100              -0.489913
                                96                  101              -0.220612
                                92                   98               0.106948
                                93                   99               0.106804
                                93                  100               0.060256
                                94                  103               0.055970
                                96                  104               0.054956
            15   2.33647 eV
                                92                   97               0.724443
                                94                  100               0.429112
                                92                   98              -0.373519
                                95                  101               0.347479
                                92                  101              -0.075121
                                93                   99              -0.070022
                                96                  104              -0.061882
                                93                  100              -0.061615
                                96                  101              -0.059893
                                91                   97              -0.052949
            16   2.38349 eV
                                96                  102              -0.633398
                                93                   98              -0.536787
                                93                   97              -0.335715
                                96                  100               0.178841
                                95                   99              -0.166145
                                95                  100               0.138687
                                95                  102              -0.125967
                                94                   98              -0.123547
                                94                   97               0.121914
                                96                  103               0.115691
                                92                   99              -0.093640
                                96                   99               0.093367
                                90                   97               0.090983
                                92                  100              -0.070092
                                90                   98               0.064769
                                95                  105              -0.052151
                                96                  106               0.050923
            17   2.46109 eV
                                94                  100               0.615904
                                92                   98               0.477414
                                93                   99               0.464173
                                95                  101              -0.292503
                                96                  104              -0.219931
                                93                  100               0.099478
                                96                  101              -0.074603
                                94                   99               0.072615
                                92                   97               0.050440
            18   2.57865 eV
                                93                   99               0.655993
                                95                  101               0.445581
                                94                  100              -0.339582
                                91                   97              -0.290505
                                96                  101               0.232325
                                92                   98               0.164289
                                94                  103              -0.138159
                                92                   97               0.117256
                                93                  100              -0.110944
                                94                   99              -0.083765
                                92                  101              -0.063389
                                91                  101              -0.063152
            19   2.61095 eV
                                94                  101               0.770437
                                96                  103              -0.345436
                                96                  102              -0.331766
                                92                   99               0.279367
                                93                  101               0.143531
                                95                  100              -0.139843
                                90                   97              -0.103506
                                96                  100              -0.081184
                                94                   98               0.076341
                                95                  103              -0.075876
                                93                   98               0.064982
                                95                  102              -0.057837
                                91                   99               0.055702
                                96                  105               0.052344
            20   2.69513 eV
                                95                  102               0.878679
                                96                  102              -0.277341
                                92                   99               0.225962
                                90                   98               0.148050
                                95                  103               0.123489
                                94                  101              -0.107757
                                96                  103               0.082767
                                93                   97               0.072802
                                92                  100              -0.067978
                                94                  104              -0.057755
                                93                   98               0.053642
 -------------------------------------------------------------------------------
 
```

