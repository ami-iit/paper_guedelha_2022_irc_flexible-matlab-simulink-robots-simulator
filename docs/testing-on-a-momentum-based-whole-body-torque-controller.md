## Testing on a Momentum-based Whole-body Torque Controller

This set of experiments tests the simulator on a controller performing a complex trajectory. For this purpose we configured the simulator library `RobotDynWithContacts` to emulate the iCub humanoid robot model with 23 degrees of freedom and integrated it with a momentum-based whole-body torque controller[^1].

[^1]: This relates to Section III - C of the paper.

The one line installation installed the resulting Simulink model example in `<install-path>/robotology-matlab/mex/+wbc/examples/+floatingBaseBalancingTorqueControlWithSimulator`, that we shall refer to as `<model-parent-folder>`, but you can then open the model from the MATLAB command line, from any directory, as follows:
```
floatingBaseBalancingTorqueControlWithSimulator.torqueControlBalancingWithSimu
```

You can find the original model and the respective documentation in the repository `whole-body-controllers`, [here]](https://github.com/robotology/whole-body-controllers/tree/master/controllers/%2BfloatingBaseBalancingTorqueControlWithSimulator).

The controller task balances the robot on a single foot while performing dynamic motions with the arms and the free leg.

### MATLAB Version Compatibility

The Simulink model `torqueControlBalancingWithSimu.mdl` was generated with Matlab R2020b.

### Supported Robots

Currently, the only supported robot model is `iCubGazeboV2_5` which has the proper modified inertia of the intermediate small and light links within the 3-DoF joints (shoulder pitch-roll-yaw, hip pitch-roll-yaw, etc), tuned for stabilising the dynamics of the simulation. There is no direct link with Gazebo, except that it was initially used on Gazebo before the simulator `matlab-whole-body-simulator` was implemented and deployed in the lab.

### Running the Simulation and Profilers

We describe in this section the steps to follow in the MATLAB/Simulink environment in order to run the simulation and profilers and read the total execution time.

In the steps below we assume you have followed and completed the steps in the [One Line Installation](../README.md#floppy_disk-one-line-installation) steps of the main README.

1. Set the `YARP_ROBOT_NAME` environment variable to `iCubGazeboV2_5`.
    ```
    >> setenv('YARP_ROBOT_NAME','iCubGazeboV2_5')
    ```

1. Leave the default configuration parameters `robot_config`, `contact_config` and `physics_config` defined in file `<model-parent-folder>/app/robots/iCubGazeboV2_5/configRobotSim.m` ("input motor reflected inertia format" default value is set to `matrix`).

1. :warning: The IMU frame name changed in the model URDF file [robotology/icub-models/iCub/robots/iCubGazeboV2_5/model.urdf, commit **72e2aa5**](https://github.com/robotology/icub-models/blob/72e2aa5ecafefac3c99aac8cfdd86118def87a72/iCub/robots/iCubGazeboV2_5/model.urdf). The frame name changed from `imu_frame` to `head_imu_0`, for a matter of naming consistency across models, considering that some models have more IMUs installed (e.g. IMU frame `chest_imu_0` on iCub3 chest). Update the IMU frame name variables `Frames.IMU` and `FramesSim.IMU` respectively in files `configRobot.m` and `configRobotSim.m`  in folder `<model-parent-folder>/app/robots/iCubGazeboV2_5`.

1. open the main config initialisation file `initTorqueControlBalancingWithSimu.m` as follows:
   ```
   >> open floatingBaseBalancingTorqueControlWithSimulator.initTorqueControlBalancingWithSimu
   ```

1. In the same config file:
    - set `Config.SIMULATION_TIME` to `45` (seconds),
    - check that `DEMO_TYPE` is set to `YOGA`.

1. In the visualizer configuration file `<model-parent-folder>/src/initVisualizer.m`, turn off the visualizer: `confVisualizer.visualizeRobot = false`.

1. open the model from the MATLAB command line, from any directory, as follows:
   ```
   >> floatingBaseBalancingTorqueControlWithSimulator.torqueControlBalancingWithSimu
   ```

1. Run the Simulink profiler and model From the Simulink window:
    - select the "Debug" tab on the Toolstrip,
    - click the "Performance Advisor" drop-down and select "Simulink Profiler", this opens a new tab "PROFILE",
    - in that same tab, hit the "Profile" button to run the model with profiling enabled.

1. The Simulink profiler stops automatically after the simulation ends and displays the profile report in a lower pane of the "PROFILE" tab, as shown in the example below[^2]:

![image](https://user-images.githubusercontent.com/6848872/234770146-0ef8989e-80bd-4f75-aed0-b414130fd5b2.png)

The total simulator running time can be seen in the second time entry ("Robot Simulator") of the report.

[^2]: Example produced on a machine with Apple M1 Max (8 performance cores and 2 efficiency cores) CPU and 32(GB) RAM, on MATLAB R2020b, so with lower RAM, MATLAB not optimised for M1 CPUs, delivering less performance than the machine used for the paper.
