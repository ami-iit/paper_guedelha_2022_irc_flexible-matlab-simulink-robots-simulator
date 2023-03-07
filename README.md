<h1 align="center">
A Flexible MATLAB/Simulink Simulator for Robotic Floating-base Systems in Contact with the Ground
</h1>
<div align="center">
<i>
N. Guedelha, V. Pasandi, G. L'Erario, S. Traversaro and D. Pucci, "A Flexible MATLAB/Simulink Simulator for Robotic Floating-base Systems in Contact with the Ground" in 2022 Sixth IEEE International Conference on Robotic Computing (IRC), Italy, 2022 pp. 53-57
</i>
</div>

<p align="center">
:construction: Work in progress (recording of the slides)
</p>

<div align="center">
  2022 Sixth IEEE International Conference on Robotic Computing (IRC)
</div>

<div align="center">
<a href="#installation-and-usage"><b>Installation and Usage</b></a> |
<a href="#reproducing-the-experiments"><b>Reproducing The Experiments</b></a> |
<a href="#results"><b>Results</b></a> |
<a href="https://ieeexplore.ieee.org/document/10023775"><b>IEEE</b></a> |
<a href="https://arxiv.org/abs/2211.09716"><b>arXiv</b></a> |
<a href="#citing-this-work"><b>Cite</b></a>
</div>

### Abstract

Physics simulators are widely used in robotics fields, from mechanical design to dynamic simulation, and controller design. This paper presents an open-source MATLAB/Simulink simulator for rigid-body articulated systems, including manipulators and floating-base robots. Thanks to MATLAB/Simulink features like MATLAB system classes and Simulink function blocks, the presented simulator combines a programmatic and block-based approach, resulting in a flexible design in the sense that different parts, including its physics engine, robot-ground interaction model, and state evolution algorithm are simply accessible and editable. Moreover, through the use of Simulink dynamic mask blocks, the proposed simulation framework supports robot models integrating open-chain and closed-chain kinematics with any desired number of links interacting with the ground. The simulator can also integrate second-order actuator dynamics. Furthermore, the simulator benefits from a one-line installation and an easy-to-use Simulink interface.

### Installation and Usage

This section describes the steps for installing the simulation framework required for reproducing the experiments depicted in the paper.
- For users not familiar with the `conda` package manager nor [Git](https://git-scm.com/) version control system, we recommend to use a **one line installer**[^1] as described in the following sections.
- Otherwise, users can: either use the [Conda package manager](https://anaconda.org) for installing directly the `conda` package [`matlab-whole-body-simulator`](https://anaconda.org/robotology/matlab-whole-body-simulator)[^2], following the guidelines in https://github.com/ami-iit/matlab-whole-body-simulator/blob/master/README.md#binary-installation-from-the-conda-robotology-channel); either install the simulator library from source, following the guidelines in https://github.com/ami-iit/matlab-whole-body-simulator/blob/master/README.md#floppy_disk-source-installation.  

[^1]: For the original fully detailed guidelines, you can refer to https://github.com/robotology/robotology-superbuild/blob/master/doc/matlab-one-line-install.md#one-line-installation-of-robotology-matlabsimulink-packages.
[^2]: Available since the `conda` build number 8 of the `conda` binaries hosted in the [robotology conda channel](https://anaconda.org/robotology).

For the complete installation guide, please refer to the documentation in simulator project repository [README.md](https://github.com/ami-iit/matlab-whole-body-simulator/blob/master/README.md).


#### :floppy_disk: One Line Installation

The one line installer can be downloaded and run from the Matlab command line, without any access to a terminal:
1. Run Matlab.
1. In the Matlab command line change the current folder to a directory where you wish to download the one installer script and install all the packages.
1. Run the following commands:
```matlab
websave('install_robotology_packages.m', 'https://raw.githubusercontent.com/robotology/robotology-superbuild/master/scripts/install_robotology_packages.m')
install_robotology_packages()
robotology_setup
```

**No install path provided:** If you do not provide any input parameter to `install_robotology_packages`, this function will install all the robotology packages related to MATLAB or Simulink in a default local directory `robotology-matlab` under the working directory, without perturbing anything else in your system:
the Yarp and iDynTree, OSQP and  Casadi MATLAB bindings, the WB-toolbox, the iCub models, the MATLAB Whole-body Controllers and the **matlab-whole-body-simulator** conda package.

**Install path provided:** You can provide an alternative installation path by calling instead `install_robotology_packages('installPrefix',<arbitrary-install-absolute-path-without-spaces>)`. If the path has spaces, use `\` to escape them.

Once the packages have been installed, you just need to re-run the `robotology_setup` script at each MATLAB restart (or add it in your [`setup.m`](https://www.mathworks.com/help/matlab/ref/startup.html) file) to make the library available again.

#### :eyes: Checking The Installation
1. After running the one-line installation and `robotology_setup`, you should be able to run the following MATLAB code without any error:
```
vec = iDynTree.Vector3();
vec.fromMatlab([1,2,3])
vec.toString();
```
1. Check the MATLABPATH environment variable. It should now have...
    ```
    <install-path>/mex: <install-path>/share/WBToolbox: <install-path>/share/WBToolbox/images
    ```
    Check the mex and Simulink libraries in the folder `<install-path>/mex`. It should contain:
    ```
    +iDynTree               BlockFactory.mexmaci64      mwbs_lib.slx
    +iDynTreeWrappers       BlockFactory.tlc            mwbs_robotDynamicsWithContacts_lib.slx
    +mwbs                   iDynTreeMEX.mexmaci64       mwbs_robotSensors_lib.slx
    +yarp.                  yarpMEX.mexmaci64           mwbs_visualizers_lib.slx
    ```
1. The `Matlab Whole-Body Simulator` library, along with the sub-libraries **robotDynamicsWithContacts**, **robotSensors** and **visualizers** should be visible in the Simulink Library Browser. They can be dragged and dropped into any open Simulink model.
<img width="963" alt="image" src="https://user-images.githubusercontent.com/6848872/116485698-1ff57580-a88c-11eb-8856-c4527e00b401.png">


### Reproducing The Experiments

We run three sets of experiments for:
- illustrating the simulation speed improvement resulting from the introduction of the Simulink Functions,
- validating the simulator with a momentum-based whole-body torque controller performing a complex trajectory,
- comparing our simulator with Gazebo,
- illustrating the flexibility of our simulator design when adding new features.

#### Analysing the Simulation Speed Improvement

The first set of experiments aims at illustrating the simulation speed improvement resulting from the introduction of the Simulink Functions.
This relates to Sections III - A & B of the paper.

#### Testing on a Momentum-based Whole-body Torque Controller

The experiment tests the simulator on controller performing a complex trajectory. For this purpose we integrated the simulator library, configured to emulate the iCub humanoid robot model with 23 degrees of freedom, with a momentum-based whole-body torque controller. The controller task balances the robot on a single foot while performing dynamic motions with the arms and the free leg.
This relates to Section III - C of the paper.

#### Comparing our Simulator with Gazebo

The third set of experiments compares our simulator with Gazebo, commonly used in the robotics community, while running a benchmark test we define for that purpose.
This relates to Section III - D of the paper.

#### Analysing the Design Flexibility

In this experiment we add a new feature to the simulation framework, namely the support of a contact model for spherical feet. The goal is to illustrate the flexibility in the framework implementation.
This relates to Section III - E of the paper.


### Results

:construction: Work in progress

### Citing this work

If you find the work useful, please cite our publication:

```
  @INPROCEEDINGS{10023775,
  author={Guedelha, Nuno and Pasandi, Venus and L’Erario, Giuseppe and Traversaro, Silvio and Pucci, Daniele},
  booktitle={2022 Sixth IEEE International Conference on Robotic Computing (IRC)}, 
  title={A Flexible MATLAB/Simulink Simulator for Robotic Floating-base Systems in Contact with the Ground}, 
  year={2022},
  volume={},
  number={},
  pages={53-57},
  doi={10.1109/IRC55401.2022.00015}}
```

### Maintainer

This repository is maintained by:

| | |
|:---:|:---:|
| [<img src="https://github.com/nunoguedelha.png" width="40">](https://github.com/nunoguedelha) | [Nuno Guedelha](https://github.com/nunoguedelha) |
| [<img src="https://github.com/VenusPasandi.png" width="40">](https://github.com/VenusPasandi) | [Venus Pasandi](https://github.com/VenusPasandi) |
