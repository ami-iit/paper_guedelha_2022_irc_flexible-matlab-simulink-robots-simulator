## Analysing the Simulation Speed Improvement

The first set of experiments aims at illustrating the simulation speed improvement resulting from the introduction of the Simulink Functions.
This relates to Sections III - A & B of the paper.

The dynamics computation functions in the class `Robot` targeted for the performance analysis are the following:
- `get_mass_matrix` computes the robot inertial matrix
- `get_bias_forces` computes the generalised bias forces,
- `get_feet_jacobians` computes the feet frames Jacobians,
- `get_feet_JDot_nu` computes the feet bias accelerations,
- `get_feet_H` computes the feet frames transformations.

For measuring the performance we’ve used two profilers, the MATLAB profiler which tracks the execution of all MATLAB interpreted code, and the Simulink profiler which tracks the execution of the Simulink blocks and the code produced by the Simulink code generator. During each performance analysis (before and after the code optimisation introducing the Simulink functions), we run both profilers for accounting for their impact on the performance and for covering all the running code.

### Running the Simulation and Profilers

We describe in this section the steps to follow in the MATLAB/Simulink environment in order to run the simulation and profilers, export the profilers output data to the workspace as `.mat` files[^1], then run the [`loadTimeSeries.m`](./scripts/loadTimeSeries.m) script for evaluating the execution times of the dynamics computation functions and generating the performance comparison plot. For the general usage instructions, refer to the docmentation in https://github.com/ami-iit/matlab-whole-body-simulator/blob/master/README.md#runner-how-to-use-the-simulator.

In the steps below we assume you have followed and completed the steps in the [One Line Installation](../README.md#floppy_disk-one-line-installation) steps of the main README.

[^1]: Stored in the folder [data](./data).


1. In a terminal, go to the directory where you cloned the repository `matlab-whole-body-simulator`and checkout the commit depending on the simulator version you wish to profile:
    - profiling before the optimisation: use commit 6af23d0d11d3b3153a374d649ce91535104eeb6f.
    - profiling after the optimisation: use commit 266512a87b6a5bcf438744463855ecbd290d520d.

1. In the MATLAB command line change the current folder to that same directory.

1. Select the robot model to `iCubGenova04` by setting the environment variable `YARP_ROBOT_NAME`: `setenv('YARP_ROBOT_NAME','iCubGenova04')`

1. Leave the default configuration parameters `robot_config`, `contact_config` and `physics_config` defined in the `./app/robots/iCubGenova04/configRobot.m` file.
<img width="900" alt="image" src="https://user-images.githubusercontent.com/34647611/159449934-c4af969b-1ee3-461c-bbe9-4b4692745085.png">

1. In the main config initialisation file `./init.m`:
    - set `Config.simulationTime` to `1` (second),
    - profiling before the optimisation: set `Config.USE_OSQP` to `false` (the MATLAB solver `quadprog` shall be used),
    - profiling after the optimisation: set `Config.USE_OSQP` to `false`, `Config.USE_QPOASES` to `true` (the qpOASES[^2] solver shall be used).

1. In this short simulation, we are feeding null torques to the iCub robot model, so you can leave the parameter "input motor reflected inertia format" set to `vector`.

<img width="470" alt="image" src="https://user-images.githubusercontent.com/6848872/117595272-adfd1600-b140-11eb-8481-698f7b1773d9.png">

1. Open the model `./test_matlab_system.mdl`.

1. Run the MATLAB profiler from the command line:
    - launch the profiler app window which also displays profiling results and the Flame graph: `profile viewer`,
    - start the profiler from the command line: `profile on`.

1. Run the Simulink profiler and model From the Simulink window:
    - select the "Debug" tab on the Toolstrip,
    - click the "Performance Advisor" drop-down and select "Simulink Profiler", this opens a new tab "PROFILE",
    - in that same tab, hit the "Profile" button to run the model with profiling enabled.

1. The Simulink profiler stops automatically after the simulation ends.

1. Export and save the Simulink profiler results into a `<filename>.mat` file by clicking "Export to MAT" in the "PROFILE" tab. This saves the data into a variable `profilerData` in the MAT file.

1. Stop the MATLAB profiler: `profilerData_interpreter = profile('info')`. This also imports the profiler results[^3] into a new variable `profilerData_interpreter` created in the workspace.

1. Save the imported results into the same `<filename>.mat` file created by the Simulink profiler in previous steps: `save("<filename>.mat","profilerData_interpreter","-append")`.

1. For running again the simulation generating new profiling result, do:
    - turn on again the MATLAB profiler (`profile on`) and hit again the "Profile" button in the "PROFILE" tab.
    - after the simulation ends, stop the profilers and export the data to a new MAT file as described previously.
    
    Generate and save a new profile result as many times as desired, with the simulator version before the optimisation and the version after the optimisation. Figure 3 bar graph in the paper was generated from all six reports in the folders `./data/profilerResults_before_optim` and `./data/profilerResults_after_optim`. Profile result file names always follow the pattern `test_matlab_system_*.mat`. If you change that pattern or folder name, remember to reflect it in the script `./loadTimeSeries`.

1. Run the script [scripts/loadTimeSeries.m](./scripts/loadTimeSeries.m) for generating and plotting the targeted functions execution times.

[^2]: For any further details on this solver please refer to https://github.com/coin-or/qpOASES.
[^3]: The generated report in the GUI window provides the number of calls, the self time (which excludes child functions execution time) and the total time. The same data is accessible through the imported results.

### On the Profilers and Profiling Script

#### Profiling before the Code Optimisation
- The Simulink profiler tracks the total execution times of the most outer Simulink block `RobotDynWithContacts` and the MATLAB System block within, implemented by the class `step_block`. The later defines methods `setupImpl` and `stepImpl` which respectively instantiate the class `Robot` and call its dynamics computation functions that we wish to track. The MATLAB System block `step_block` total execution time includes the total execution time of `stepImpl` (that of `setupImpl` being negligible).
- The MATLAB profiler tracks the execution of all MATLAB interpreted code called from `stepImpl`, including the class `Robot` dynamics computation functions mentioned earlier (`get_mass_matrix`, `get_bias_forces`, `get_feet_jacobians`, `get_feet_JDot_nu`, `get_feet_H`)[^4].

[^4]: These functions run a cascade of wrapper functions that call MEX functions built with C Matrix API, and which implement the iDynTree library functions.

- **Tracking the dynamics computation functions individually:** The MATLAB profiler tracks the execution of all MATLAB interpreted code called from the class `Robot` targeted functions listed above. That code runs a cascade of wrapper functions that call MEX functions built with C Matrix API, and which implement the iDynTree library functions.
- **Tracking the total execution time:** The Simulink profiler tracks the total execution time of the most outer Simulink block `RobotDynWithContacts` and the execution self-time of the MATLAB System block within, binded to the class "step_block" which instantiates the class `Robot` and calls the dynamics computation functions. Note that as the Simulink profiler tracks only the Step MATLAB System execution self-time, not the MATLAB interpreted code sub-calls, we characterize the core-simulator total execution time by the Simulink block `RobotDynWithContacts` execution time.

#### Profiling after the Code Optimisation:

- **Tracking the dynamics computation functions individually:** The MATLAB profiler cannot track the code produced by the Simulink code generator, so we can count only on the Simulink profiler output for the targeted functions.
- **Tracking the total execution time:** The Simulink profiler tracks the execution of the most outer Simulink block `RobotDynWithContacts`, the Step MATLAB System block within, and the Simulink functions (Simulink blocks triggered programmatically from the dynamics computation functions in the class `Robot`).
