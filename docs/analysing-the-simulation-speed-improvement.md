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

### Before the code optimisation:

- The Simulink profiler tracks the execution of the most outer Simulink block `RobotDynWithContacts` and the Step MATLAB System block within, which which instantiates the class `Robot` and calls the dynamics computation functions. The Simulink profiler tracks only the Step MATLAB System execution self-time, not the MATLAB interpreted code sub-calls, so the core-simulator total execution time is characterised by the Simulink block `RobotDynWithContacts` execution time.
- The MATLAB profiler cannot track the code produced by the Simulink code generator, so we can count only on the Simulink profiler output for the targeted functions.

### After the code optimisation:

- The MATLAB profiler tracks the execution of all MATLAB interpreted code called from the class `Robot` targeted functions listed above. That code runs a cascade of wrapper functions that call MEX functions built with C Matrix API, and which implement the iDynTree library functions.
- The Simulink profiler tracks the execution of the most outer Simulink block `RobotDynWithContacts`, the Step MATLAB System block within, and the WBT Simulink blocs implements a C++ abstraction layer wrapping the iDynTree dynamics library through the BlockFectory framework the dynamics computation functions. The Simulink profiler tracks only the Step MATLAB System execution self-time, not the MATLAB interpreted code sub-calls, so the core-simulator total execution time is characterised by the Simulink block `RobotDynWithContacts` execution time.

The script generating and plotting the targeted functions execution times can be found in the folder [scripts/loadTimeSeries.m](./scripts/loadTimeSeries.m).

The profilers output data can be found in the folder [data](./data).
