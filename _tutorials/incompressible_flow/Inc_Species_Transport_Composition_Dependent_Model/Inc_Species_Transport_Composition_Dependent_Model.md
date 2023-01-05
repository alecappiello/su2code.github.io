---
title: Composition-Dependent model for Species Transport equations
permalink: /tutorials/Inc_Species_Transport_Composition_Dependent_Model/
written_by: Cristopher-Morales 
for_version: 7.5.0
revised_by:  
revision_date:
revised_version:
solver: INC_RANS
requires: SU2_CFD
complexity: intermediate
follows: Inc_Species_Transport
---


## Goals

In this tutorial, the user will be familiarized with the composition-dependent model in SU2 based on the Ideal Gas law for a gas mixture. The necessary steps and configuration options for the aforementioned model will be explained through a 3D incompressible kenics static mixer for a methane-air mixture.

## Resources

The resources for this tutorial can be found in the [incompressible_flow/Inc_Species_Transport_Composition_Dependent_Model](https://github.com/su2code/Tutorials/tree/master/incompressible_flow/Inc_Species_Transport_Composition_Dependent_Model) directory in the [tutorial repository](https://github.com/su2code/Tutorials). In order to complete this tutorial, you will need the configuration file ([kenics_mixer_tutorial.cfg](https://github.com/su2code/Tutorials/tree/master/incompressible_flow/Inc_Species_Transport_Composition_Dependent_Model/kenics_mixer_tutorial.cfg)) and the mesh file ([kenics.su2](https://github.com/su2code/Tutorials/tree/master/incompressible_flow/Inc_Species_Transport_Composition_Dependent_Model/kenics.su2)).

The mesh is created using [gmsh](https://gmsh.info/) and a respective `.geo` script is available to recreate/modify the mesh [kenics_mixer_tutorial.geo](https://github.com/su2code/Tutorials/tree/master/incompressible_flow/Inc_Species_Transport_Composition_Dependent_Model/kenics_mixer_tutorial.geo). The mesh is fully structured (i.e. only contains Quadrilateral elements) with 3364 elements and 3510 points.

![Mesh with boundary conditions](../../tutorials_files/incompressible_flow/Inc_Species_Transport_Composition_Dependent_Model/images/mesh.jpg)
Figure (1): Computational mesh with color indication of the used boundary conditions.

## Prerequisites

The following tutorial assumes you have already compiled `SU2_CFD` in serial or parallel, please see the [Download](/docs_v7/Download/) and [Installation](/docs_v7/Installation/). Likewise, it is advised to perform the tutorial regarding species transport in a venturi mixer([Inc_Species_Transport](/tutorials/Inc_Species_Transport/)) where the species transport options are explained, as this tutorial is mainly focused on the composition-dependent options.

## Background

The geometry consists on a Static Kenics mixer with three blades, each blade starts at 90 degrees perpendicular to the previous blades and they are being twisted 180 degrees along the z-axis in order to enhance the mixing. Furthermore, two inlets are considered where pure air and pure methane are injected at each inlet. Finally, one oulet is considered at the end of the mixer device in order to study the pressure drop and the variance of the mass fraction, which is a measure of the mixing performance of the mixing device.


## Problem Setup

In this problem, we study the flow and mixing properties along the static kenic mixer. Then, we have the following boundary conditions at the inlets and outlet:

- Equal Inlet Velocities (constant) = 5 m/s in normal direction (z-direction)
- Outlet Pressure (constant) = 0 Pa
- Inlet Temperature (both inlets) = 300 K
- Adiabatic walls.

The SST turbulence model is used with the default settings of freestream turbulence intensity of 5% and a turbulent-to-laminar viscosity ratio of 10. However, in order to highlight the new option available in SU2 of having different turbulence intensities and turbulent-to-laminar viscosity ratios, these will be given as marker inlet turbulent that will be explained in the following section.

The thermochemical properties for each gas are given below:
* Methane:
    - Molecular Weight = 16.043 [g / mol]
    - Viscosity: 1.1102E-05 [kg /(m s)]
    - Heat capacity at constant pressure: 2224.43 [J/(kg K)]
    - Thermal Conductivity: 0.0357 [W /(m K)]
* Air: 
    - Molecular Weight = 28.960 [g / mol]
    - Viscosity: 1.8551E-05 [kg /(m s)]
    - Heat capacity at constant pressure: 1009.39 [J/(kg K)]
    - Thermal Conductivity: 0.0258 [W /(m K)]

The species mass fractions at each inlet are the following:

- Inlet_1: mass fractions methane, Y_CH4 = 1.0 (pure methane, Y_air=0.0)
- Inlet_2: mass fractions methane, Y_CH4 = 0.0 (pure air, Y_air=1.0)

It must be noticed that inside SU2, for a mixture of N species, N-1 species transport equations are being solved and the last species is computed as $1-\sum_{i=1}^{N-1} Y_i$. Thus, in this tutorial, a transport equation for methane is being solved. For more information, please see [Theory](/docs_v7/Theory/).

## Configuration File Options

All available options concerning species transport are listed in the [config_template.cfg](https://github.com/su2code/SU2/blob/master/config_template.cfg).Here, we are going to focus in the composition-dependent options.

For activating the composition-dependent model, the fluid model must be chosen as `FLUID_MODEL= FLUID_MIXTURE`. It must be noted that this model is only compatible with `INC_DENSITY_MODEL= VARIABLE`. Otherwise, an error will be shown at run-time.

For incompressible flows, a low-mach Number approximation allows to decompose the pressure into dynamics and thermodynamics (operating) pressure (see [Theory]/docs_v7/Theory/). The operating pressure is used for computing the mixture density using the Ideal gas law. The thermodynamic pressure might strongly affects the density at the inlets causing unphysical results. Therefore, the thermodynamic pressure must be provided for the user for the `FLUID_MIXTURE` model and it is not longer computed from the free-stream conditions as it is done in the other fluid models. As in mixing and combustion processes, the operating pressure is often assumed as 101325 pa, then this is the default value considered inside SU2 if the thermodynamics pressure is not given in the .cfg file. The thermodynamics pressure is given in the .cfg file as follow: `THERMODYNAMIC_PRESSURE= 101325.0`.

Subsequently, The molecular weights and Heat capacities at constant pressure must be provided as a list as follow: `MOLECULAR_WEIGHT= W_1, W_2,...., W_N` ,  `SPECIFIC_HEAT_CP = Cp_1, Cp_2,..., Cp_N`. The length of the list must match the number of the N species in the mixture. Moreover, the mean molecular weight is computed as a mole fraction average and the mixture heat capacity is computed as a mass fraction average. For more information, please see $^{1},^{3}$.

For the conductivity model, the following options are available: `CONDUCTIVITY_MODEL= CONSTANT_CONDUCTIVITY, CONSTANT_PRANDTL, POLYNOMIAL_CONDUCTIVITY `. In this tutorial, the option `CONSTANT_CONDUCTIVITY` is used. For this option, a constant conductivity for each species must be provided as follow: `THERMAL_CONDUCTIVITY_CONSTANT= k_1, k_2,...., k_N`. 
Currently, the only mixing law available in SU2 for computing the mixture thermal conductivity is based on the Wilke mixing law. Therefore, this is the default option and it is hardcoded for the `FLUID_MIXTURE` option. For more information regarding this mixing model, please see $^{1},^{2}$.

Similar treatment is done for the Laminar Prandtl numbers: `PRANDTL_LAM= Pr_1, Pr_2,....,Pr_N`. Finally, for turbulence simulations, the option of turbulent Prandlt number can be enabled as `TURBULENT_CONDUCTIVITY_MODEL= CONSTANT_PRANDTL_TURB`. If this option is enabled, the turbulent Prandtl numbers must follow the same structure as the Laminar Prandtl numbers: `PRANDTL_TURB= Pr_Turb_1, Pr_Turb_2, ..., Pr_Turb_N`. For more information about laminar and turbulent Prandlt Number, please see [Theory](/docs_v7/Theory/).

For the present tutorial, the options are given below:

```
% -------------------- FLUID PROPERTIES ------------------------------------- %
%
FLUID_MODEL= FLUID_MIXTURE
% Thermodynamics(operating) Pressure (101325 Pa default value, only for incompressible flow and FLUID_MIXTURE)
THERMODYNAMIC_PRESSURE= 101325.0
%
MOLECULAR_WEIGHT= 16.043, 28.960
%
SPECIFIC_HEAT_CP = 2224.43, 1009.39
%
CONDUCTIVITY_MODEL= CONSTANT_CONDUCTIVITY
THERMAL_CONDUCTIVITY_CONSTANT= 0.0357, 0.0258
%
PRANDTL_LAM= 0.72, 0.72
%
TURBULENT_CONDUCTIVITY_MODEL= CONSTANT_PRANDTL_TURB
PRANDTL_TURB= 0.90, 0.90
```

Regarding the viscosity model, the following options are available `VISCOSITY_MODEL= SUTHERLAND, CONSTANT_VISCOSITY, POLYNOMIAL_VISCOSITY`. In the case of `CONSTANT_VISCOSITY`, the viscosities must be provided as a list as follow: `MU_CONSTANT= mu_1, mu_2, ..., mu_N`. Similarly, if `SUTHERLAND` model is chosen, the sutherland parameters must be given as a list for each species in the mixture. For completeness, an example for SUTHERLAND option is given below for a mixture of two species:

```
% --------------------------- VISCOSITY MODEL ---------------------------------%
%
VISCOSITY_MODEL= SUTHERLAND
% 
MU_REF= 1.118E-05, 1.716E-05
%
MU_T_REF= 273, 273  
%
SUTHERLAND_CONSTANT= 97, 111
```

For this tutorial, as the energy equation is not being solved, we use `CONSTANT_VISCOSITY` as the viscosity model. Finally, for computing the mixture viscosity, two models are available in SU2 which are Wilke and Davidson Model. They can be enabled using the following option: `MIXING_VISCOSITY_MODEL = WILKE, DAVIDSON`. For further details about these models, please see $^{2},^{4}$.
The options used in this tutorial are shown below:

```
% --------------------------- VISCOSITY MODEL ---------------------------------%
%
VISCOSITY_MODEL= CONSTANT_VISCOSITY
%
MU_CONSTANT= 1.1102E-05, 1.8551E-05 
%
MIXING_VISCOSITY_MODEL = WILKE
```

The Species transport is switched on by setting `KIND_SCALAR_MODEL= SPECIES_TRANSPORT`. For the mass diffusivity, the following models are available `DIFFUSIVITY_MODEL= CONSTANT_DIFFUSIVITY, CONSTANT_SCHMIDT, UNITY_LEWIS, CONSTANT_LEWIS` , where `CONSTANT_DIFFUSIVITY` is the default model. For the two first, a constant value for all species must be given in the .cfg file, as it is done in the species transport tutorial [Inc_Species_Transport](/tutorials/Inc_Species_Transport/). For the UNITY_LEWIS, no values must be provided because the diffusivity is computed using the mixture thermal conductivity, density and heat capacity at constant pressure, for more information please see $^{3}$. For highly diffusive gases, such as hydrogen, the `CONSTANT_LEWIS` option could be used. For this option, the Lewis numbers of the N-1 species which a transport equation is being solved must be provided as a list using the following option `CONSTANT_LEWIS_NUMBER= Le_1, Le_2, ..., Le_N_1`. Finally, for turbulent simulations, the turbulent diffusivity is computed based on the `SCHMIDT_NUMBER_TURBULENT`. For reference, please consult [the respective theory](/docs_v7/Theory/#species-transport).

Finally, for the SST model, it is possible to provide the intensity and turbulent-to-laminar viscosity ratios per inlet. For this option, we use the following structure `MARKER_INLET_TURBULENT= (inlet_1, TurbIntensity_1, TurbLamViscRatio_1, inlet_2, TurbIntensity_2, TurbLamViscRatio_2, ...)`. The other species transport options can be found in the species transport. 

The number of species transport equations is not set individually but deduced from the number of values given in the respective lists for species options. SU2 checks whether the same amount of values is given in each option and solves the appropriate amount of equations. `MARKER_INLET_SPECIES` is one of these options and has to be used alongside a usual `MARKER_INLET`. For outlets, symmetries or walls this is not necessary. 

The option `SPECIES_USE_STRONG_BC` should be left to `NO` and is an experimental option where a switch to strongly enforced boundary conditions can be made.

For `CONV_NUM_METHOD_SPECIES= SCALAR_UPWIND` a second order MUSCL reconstruction and multiple limiters are available.

The `TIME_DISCRE_SPECIES` can be either an implicit or explicit euler and a CFL reduction coefficient `CFL_REDUCTION_SPECIES` compared to the regular `CFL_NUMBER` is available.

The inital species mass fractions are given by the list `SPECIES_INIT= 1.0, ...`.

`SPECIES_CLIPPING= YES` with the respective lists for min and max enforces a strict lower and upper limit for the mass fraction solution used by the solver.

```
% --------------------- SPECIES TRANSPORT SIMULATION --------------------------%
%
% Specify scalar transport model (NONE, SPECIES_TRANSPORT)
KIND_SCALAR_MODEL= SPECIES_TRANSPORT
%
% Mass diffusivity model (CONSTANT_DIFFUSIVITY)
DIFFUSIVITY_MODEL= UNITY_LEWIS
%
% Turbulent Schmidt number of mass diffusion
SCHMIDT_NUMBER_TURBULENT= 0.7
%
% Inlet Species boundary marker(s) with the following format:
% (inlet_marker, Species1, Species2, ..., SpeciesN-1, inlet_marker2, Species1, Species2, ...)
MARKER_INLET_SPECIES= (inlet_1,1.0, inlet_2, 0.0)
%
% Use strong inlet and outlet BC in the species solver
SPECIES_USE_STRONG_BC= NO
%
% Convective numerical method for species transport (SCALAR_UPWIND, BOUNDED_SCALAR)
CONV_NUM_METHOD_SPECIES= BOUNDED_SCALAR
%
% Monotonic Upwind Scheme for Conservation Laws (TVD) in the species equations.
% Required for 2nd order upwind schemes (NO, YES)
MUSCL_SPECIES= NO
%
% Slope limiter for species equations (NONE, VENKATAKRISHNAN, VENKATAKRISHNAN_WANG, BARTH_JESPERSEN, VAN_ALBADA_EDGE)
SLOPE_LIMITER_SPECIES = NONE
%
% Time discretization for species equations (EULER_IMPLICIT, EULER_EXPLICIT)
TIME_DISCRE_SPECIES= EULER_IMPLICIT
%
% Reduction factor of the CFL coefficient in the species problem
CFL_REDUCTION_SPECIES= 0.5
%
% Initial values for scalar transport
SPECIES_INIT= 1.0
%
% Activate clipping for scalar transport equations
SPECIES_CLIPPING= NO
%
% Maximum values for scalar clipping
SPECIES_CLIPPING_MAX= 1.0, ...
%
% Minimum values for scalar clipping
SPECIES_CLIPPING_MIN= 0.0, ...
```

For the screen, history and volume output multiple straight forward options were included. Whenever a number is used at the end of the keyword, one for each species (starting at zero) can be added.
```
SCREEN_OUTPUT= RMS_SPECIES_0, ..., MAX_SPECIES_0, ..., BGS_SPECIES_0, ..., \
               LINSOL_ITER_SPECIES, LINSOL_RESIDUAL_SPECIES, \
               SURFACE_SPECIES_0, ..., SURFACE_SPECIES_VARIANCE
```

For `HISTORY_OUTPUT` the residuals are included in `RMS_RES` and the linear solver quantities in `LINSOL`. The surface outputs can be included with `SPECIES_COEFF` or `SPECIES_COEFF_SURF` for each surface individually.

For `VOLUME_OUTPUT` no extra output field has the be set. The mass fractions are included in `SOLUTION` and the volume residuals in `RESIDUAL`.

All available output can be printed to screen using the `dry-run` feature of SU2:
```
$ SU2_CFD -d <config-filename>.cfg
```

## Running SU2

The simulation can be run in serial using the following command:
```
$ SU2_CFD species3_primitiveVenturi.cfg
```
or in parallel with your preferred number of cores (for this small case not more than 4 cores should be used):
```
$ mpirun -n <#cores> SU2_CFD species3_primitiveVenturi.cfg
```

## Results

The case converges nicely as expected on such a simple case and mesh.

![Residual plot](../../tutorials_files/incompressible_flow/Inc_Species_Transport/images/residuals_specMix.png)
Figure (2): Residual plot (Incompressible mean flow, SST turbulence model, species transport).

Note that there is still some unphysical mass fraction fluctuation for Species_0 at the junction corner. This becomes much less apparent by using `MUSCL_SPECIES = YES` but does not fully disappear.

![Species Mass Fractions](../../tutorials_files/incompressible_flow/Inc_Species_Transport/images/speciesMassFractions.jpg)
Figure (3): Volume mass fractions for both species. Species_1 is mirrored for better comparison.

Velocity magnitude field along which the species are transported. For a much less homogenous mixture at the outlet one could decrease the `DIFFUSIVITY_CONSTANT` which makes for a more interesting optimization problem.

![Velocity Magnitude](../../tutorials_files/incompressible_flow/Inc_Species_Transport/images/VelocityMag.jpg)
Figure (4): Velocity Magnitude in the domain.


### References
$^{1}$ B. Poling, J. Prausnitz, J. O’Connell, The Properties of Gases and Liquids, 5th Edition, McGraw-Hill Education,2000.(URL https://books.google.nl/books?id=9tGclC3ZRX0C)

$^{2}$ C. R. Wilke, A viscosity equation for gas mixtures, The Journal of Chemical Physics 18 (4) (1950),517–519.(https:doi:10.1063/1.1747673).

$^{3}$ T. Poinsot, D. Veynante, Theoretical and Numerical Combustion, Ch. 1, 2012.

$^{4}$ T. A. Davidson, A simple and accurate method for calculating viscosity of gaseous mixtures. (URL https://www.osti.gov/biblio/6129940)


## Additional remarks

An in depth optimization of this case with addition of the FFD-box, gradient validation and some more steps can found [here](/tutorials/Species_Transport/).