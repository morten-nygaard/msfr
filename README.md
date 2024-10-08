# MSFR_OpenMC

# Model and project description
This repository contains an OpenMC model of the EVOL reference MSFR as described in [2] and the code for simulating and plotting the main results for the DTU student project in [1]. The project aims to benchmark simulation results from [3] regarding the temperature dependency of the reactivity and the temperature coefficients of reactivity in the range of 300K-1800K. These temperature maps are needed to avoid the computational load of calculating the feedback directly for each step, e.g., for design optimization. 

The results confirm the roughly linear behavior of the total reactivity in the operating temperature range of the EVOL reference MSFR of 900K-1500K, where the total temperature coefficient was found to be approximately constant: alpha = -7.4+/-0.1 pcm/K in [1] (using OpenMC, ENDF-B.7.1), corresponding to a 2.4% difference from alpha = −7.58+/-0.05 pcm/K in [3] (using Serpent, Nuclear library not stated, possibly ENDF-B.6 or JEFF-3.1 as deduced from [2] and [3]). The highest resolution achieved was #particles=300000 and #active_batches=110 due to limited computing power, and the difference fell with increasing resolution. Thus, higher resolution might further minimize the difference in alpha.

The reactivity and the temperature coefficient were split into contributions from Doppler broadening and density changes, which is necessary for design optimization since they display different feedback distributions over the core. The Doppler contribution was simulated by changing the material temperature in steps of 100K from 300K-1800K while holding the temperature to calculate the density constant at the benchmark temperature of 900K. Vice versa for the density temperature. Note that the Doppler contribution, therefore, includes all non-density-related temperature effects, such as the hardening of the neutron spectrum. A reference simulation was conducted, changing the temperatures together and showing similar results as the sum of the components. alpha = drho/dT was found through numerical differentiation.

[1] "Impacts of temperature feedback on reactivity parameters in the Molten Salt Fast Reactor" (June 2024) by Morten Nygaard, master's student at DTU Engineering Physics.

[2] "Neutronic benchmark of the molten salt fast reactor in the frame of the EVOL and MARS collaborative projects" (2019) by Mariya Brovchenko et al.

[3] "Unmoderated Molten Salt Reactors design optimization for power stability" (2022) by Axel Laureau et al.

# Description of Scripts
- "materials.py" and "geometry.py" are functions called by the run file "k_eff_loop.py" to build the materials and geometry of the recreated OpenMC model of the EVOL reference MSFR. "materials.py" takes as inputs a material temperature (for the OpenMC material temperatures of all core materials allowing for simulation of the Doppler reactivity) and a density temperature (to calculate the density of the molten salts in the fuel and fertile blanket regions allowing for simulation of the density reactivity) and exports the .xml file, and outputs the materials. "geometry.py" takes these materials as inputs, exports the .xml, and outputs the geometry.

- "k_eff_loop.py" calls "geometry.py" and "materials.py" to build the model and runs OpenMC a total of 1+3x16=49 times:
  - 1 eigenvalue simulation at the benchmark temperature of 700C.
  - 16 with T going from 300K-1800K in steps of 100K.
  - 16 with the density temperature going from 300K-1800K in steps of 100K, with material temperature fixed at 900K, giving the density reactivity.
  - 16 with the material temperature going from 300K-1800K in steps of 100K, with density temperature fixed at 900K, giving the Doppler reactivity.

Data is saved in files "data/k_eff_XXXXX.txt" with columns:
{particles}, {batches}, {inactive_cycles}, {material_temp}, {density_temp}, {k_combined}
OBS: Update "filename" manually to avoid overwriting data.

- "parameter_plot.py" reads "k_eff_loop_XXXXX.txt" files, calculates reactivity and temperature coefficients of reactivity and its propagated uncertainty, and plots versus temperature in kelvin. An error-weighted curve fit is also plotted. It requires latex to be installed, or lines can be outcommented. Plots are saved in "k_eff_loop_XXXXX_rho_plot.png" and "k_eff_loop_XXXXX_coef_plot.png" if enabled.

- "shannon_entropy.py" plots the Shannon entropy calculated in "k_eff_loop.py". The entropy converges before batch 10, so 20 inactive batches were used.

- "geometry_plot.py" creates cross-sectional plots of the model geometry and saves them as "plot_xy.png," "plot_xz.png," and "plot_yz.png."

Contact information: s182487@dtu.dk
