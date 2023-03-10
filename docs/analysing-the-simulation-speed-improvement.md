## Analysing the Simulation Speed Improvement

The first set of experiments aims at illustrating the simulation speed improvement resulting from the introduction of the Simulink Functions.
This relates to Sections III - A & B of the paper.

The dynamics computation functions in the class `Robot` targeted for the performance analysis are the following:
- `get_mass_matrix` computes the robot inertial matrix
- `get_bias_forces` computes the generalised bias forces,
- `get_feet_jacobians` computes the feet frames Jacobians,
- `get_feet_JDot_nu` computes the feet bias accelerations,
- `get_feet_H` computes the feet frames transformations.

For measuring the performance we’ve used two profilers, the MATLAB profiler which tracks the execution of all MATLAB interpreted code, and the Simulink profiler which tracks the execution of the Simulink blocks. During each performance analysis (before and after the code optimisation introducing the Simulink functions), we run both profilers for accounting for their impact on the performance and for covering all the running code.

### Running the Simulation and Profilers

We describe in this section the steps to follow in the MATLAB/Simulink environment in order to run the simulation and profilers, export the profilers output to the workspace as `.mat` files, then run the [`loadTimeSeries.m`](./scripts/loadTimeSeries.m) script for computing the dynamics computation functions execution times and generating the performance comparison plot. For the general usage instructions, refer to the docmentation in https://github.com/ami-iit/matlab-whole-body-simulator/blob/master/README.md#runner-how-to-use-the-simulator.

In the steps below we assume you have followed and completed the steps in the [One Line Installation](../README.md#floppy_disk-one-line-installation) steps of the main README.

The script generating and plotting the targeted functions execution times can be found in the folder [scripts/loadTimeSeries.m](./scripts/loadTimeSeries.m).

The profilers output data can be found in the folder [data](./data).

#### Profiling before the Code Optimisation:

- The Simulink profiler tracks the execution of the most outer Simulink block `RobotDynWithContacts` and the Step MATLAB System block within, which which instantiates the class `Robot` and calls the dynamics computation functions. The Simulink profiler tracks only the Step MATLAB System execution self-time, not the MATLAB interpreted code sub-calls, so the core-simulator total execution time is characterised by the Simulink block `RobotDynWithContacts` execution time.
- The MATLAB profiler cannot track the code produced by the Simulink code generator, so we can count only on the Simulink profiler output for the targeted functions.

1. In a terminal, go to the directory where you cloned the repository `matlab-whole-body-simulator`and checkout the commit .

1. In the MATLAB command line change the current folder to the that same directory where you cloned the `matlab-whole-body-simulator`.

1. Select the robot model to `'iCubGenova04'` by setting the environment variable `YARP_ROBOT_NAME`: `setenv('YARP_ROBOT_NAME','iCubGenova04')`

1. Leave the default configuration parameters `robot_config`, `contact_config` and `physics_config` defined in the `./app/robots/iCubGenova04/configRobot.m` file.
<img width="900" alt="image" src="https://user-images.githubusercontent.com/34647611/159449934-c4af969b-1ee3-461c-bbe9-4b4692745085.png">

1. In the main config initialisation file `./init.m`:
    - set `Config.USE_OSQP` to `false` (the MATLAB solver `quadprog` shall be used),
    - set `Config.simulationTime` to `1` (second).

1. In this short simulation, we are feeding null torques to the iCub robot model, so you can leave the parameter "input motor reflected inertia format" set to `vector`.

<img width="470" alt="image" src="https://user-images.githubusercontent.com/6848872/117595272-adfd1600-b140-11eb-8481-698f7b1773d9.png">

1. Open the model `./test_matlab_system.mdl`.

1. Launch the MATLAB profiler GUI window from the command line: `>> profile viewer`.

1. Generate a profile through the following steps: (1) turn the profiler on; (2) run the model or script; (3) stop the model; (4) turn the profiler off; (5) import the profiler results: `>> p = profile('info')`; (6) save the results to a `.mat` file: `>> save matFileName`.

The generated report in the GUI window provides the number of calls, the self time (which excludes child functions execution time) and the total time. The same data is accessible through the imported results.

1. Generate a new profile result as many times as desired. Figure 3 bar graph in the paper was generated from three reports `./data/profilerResults_before_optim/test_matlab_system_19__22-Sep-2022_02_15_33.mat`, `./data/profilerResults_before_optim/test_matlab_system_19__22-Sep-2022_08_50_43.mat`, `./data/profilerResults_before_optim/test_matlab_system_19__22-Sep-2022_09_00_30.mat`. Profile result file names always follow the pattern `test_matlab_system_*.mat`. If you change that pattern or folder name, remember to reflect it in the script `./loadTimeSeries`.

1. Run the script [scripts/loadTimeSeries.m](./scripts/loadTimeSeries.m) for generating and plotting the targeted functions execution times.


### Profiling after the Code Optimisation:

- The MATLAB profiler tracks the execution of all MATLAB interpreted code called from the class `Robot` targeted functions listed above. That code runs a cascade of wrapper functions that call MEX functions built with C Matrix API, and which implement the iDynTree library functions.
- The Simulink profiler tracks the execution of the most outer Simulink block `RobotDynWithContacts`, the Step MATLAB System block within, and the WBT Simulink blocs implements a C++ abstraction layer wrapping the iDynTree dynamics library through the BlockFectory framework the dynamics computation functions. The Simulink profiler tracks only the Step MATLAB System execution self-time, not the MATLAB interpreted code sub-calls, so the core-simulator total execution time is characterised by the Simulink block `RobotDynWithContacts` execution time.
