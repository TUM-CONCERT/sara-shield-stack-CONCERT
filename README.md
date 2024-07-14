# sara-shield-stack-CONCERT

## Overview 

[CONCERT](https://concertproject.eu/overview) is an EU project focussing on novel, configurable robots. TUM provides verifiable-safe algorithms to enable Human-robot interactions. This is mainly done using [SaRA](https://github.com/Sven-Schepp/SaRA), which calculates whether capsules around human parts intersect with a robot. These capsules grow depending on the human measurement uncertainty and frequency.
The sara-shield is integrated over [xbot2](https://advrhumanoids.github.io/xbot2/) with robots. It can be used either in simulation or on a real robot. More information about the real Robot setup can be found [here](https://github.com/TUM-CONCERT/sara-shield-stack-CONCERT/blob/main/RealRobotTest.md).

This Repository contains the Docker files for CONCERT Integration of TUM

### Related Git Repositories
 - This repo: Build instructions and Docker file, as well as Documentation,
 - [concert_description](https://github.com/ADVRHumanoids/concert_description): Includes all concert and xbot2 functionalities common for all partners.
 - [concert_msgs](https://github.com/ADVRHumanoids/concert_msgs): Defines the msgs send and received between packages. It is mainly used to define a common message that describes human poses.
 - [sara-shield-xbot2](https://github.com/TUM-CONCERT/sara-shield-xbot2): Ensures human safety by calculating reachable sets of the human and intersecting them with the robot. Takes goal joint positions or trajectories as input and outputs commands to move the robot safely. 
 - [sara_shield_forest_recipes](https://github.com/TUM-CONCERT/sara_shield_forest_recipes): Contains [forest](https://pypi.org/project/hhcm-forest/) recipes. A recipes defines how git repos are pulled, which dependencies they have and how to build the packages.
 - [human-gazebo](https://github.com/TUM-CONCERT/human-gazebo): Animated human model in gazebo sending out joint measurements.
 - [rviz_plugin_sara_shield](https://github.com/TUM-CONCERT/rviz_plugin_sara_shield) : Visualize the sara-shield status in RVIZ and send sara shield commands
 - [rviz_plugin_humans](https://github.com/TUM-CONCERT/rviz_plugin_humans): Visualize concert_msgs/Humans messages in RVIZ directly.
 - [sara_shield_utils](https://github.com/TUM-CONCERT/sara_shield_utils): Tools and Rosnodes for debugging and testing of sara-shield


### Folder structure
An extract of the most important folders (some folders removed for simplicity!):
```
tum_integration_ws
    ├── build
    │   ├── base_estimation
    │   ├── centauro_cartesio
    │   ├── eigen-3.4.0
    │   ├── realsense_gazebo_description
    │   ├── realsense_gazebo_plugin
    │   └── sara-shield
    ├── catkin_ws
    │   ├── devel
            └── setup.bash
    │   ├── setup.bash
    │   └── src
    │       ├── concert_msgs
    │       ├── sara_shield_utils
    │       ├── rviz_plugin_sara_shield
    │       ├── rviz_plugin_sara_shield
    │       └── human-gazebo
    ├── install
    │   ├── include
    │   │   ├── SaRA
    │   │   ├── base_estimation
    │   │   ├── centauro_cartesio
    │   │   └── eigen3
    │   ├── lib
    ├── recipes
    │   └── multidof_recipes
    ├── ros_src
    │   ├── concert_description -> /home/user/tum_integration_ws/src/concert_description
    │   ├── modular -> /home/user/tum_integration_ws/src/modular
    │   ├── realsense_gazebo_description -> /home/user/tum_integration_ws/src/realsense_gazebo_description
    │   └── sara-shield -> /home/user/tum_integration_ws/src/sara-shield
    └── src
        ├── base_estimation
        ├── centauro_cartesio
        ├── concert_description
        ├── eigen-3.4.0
        ├── modular
        ├── realsense_gazebo_description
        ├── realsense_gazebo_plugin
        └── sara-shield
```

# INSTALLATION

## Build the docker container
```
./build-docker.bash --no-cache 
```
Builds the image `jakobthumm/tum-concert:latest.`

There will be red lines, but this is normal. At the end of a successful build, it should say
```
Successfully built <SOME_HASH>
Successfully tagged jakobthumm/tum-concert:latest
```
Unsuccessful builds end with 
```
The command '/bin/bash -ic forest grow tum_src --verbose --jobs 4 --pwd user' returned a non-zero code: 1
```

## Run the docker container
```
./run-docker.bash
```

## Inside Docker

### Start roscore
```
roscore
```

### Start gazebo simulation
```
mon launch sara_shield concert.launch
```

### Open `xbot2-gui`
**After** Gazebo runs, start Xbot2-GUI with
```
xbot2-gui
```
This should open a large window with status "Running" in the top left corner. If it opens a small window with the status "Inactive" instead, close and rerun the command above.

### Open `rviz`
**Open Rviz** for visualization 
```
rviz -d $(rospack find sara_shield)/safety_shield/rviz/concert_sara_shield.rviz
```
Alternatively, run `rviz` and inside Rviz: go to the menu `File`, and select `Open Config` and load `/home/user/tum_integration_ws/src/sara-shield/safety_shield/rviz/concert_sara_shield.rviz`or even add the topics by hand:

The visualization topics of sara-shield are named ```/sara_shield/human_joint_marker_array``` and ```/sara_shield/robot_joint_marker_array``` and can be visualized by adding them in rviz.

and then **start the safety-shield plugin** in the xbot2 gui. 


## Send goal position:
```
rostopic pub /sara_shield/goal_joint_pos std_msgs/Float32MultiArray "layout:
  dim:
  - label: ''
    size: 0
    stride: 0
  data_offset: 0
data: [-2.5, 1.4, 0.0, 0.0, 0.0, 0.0]"
```
and then
```
rostopic pub /sara_shield/goal_joint_pos std_msgs/Float32MultiArray "layout:
  dim:
  - label: ''
    size: 0
    stride: 0
  data_offset: 0
data: [2.5, 1.4, 0.0, 0.0, 0.0, 0.0]"
```
## Send safety override
To set the safety shield to unsafe send
```
rostopic pub /sara_shield/safe_flag std_msgs/Bool "data: false"
```
send true to activate normal safety shield behavior again.

## Send dummy human measurements
Send dummy measurements:
```
rostopic pub /sara_shield/send_dummy_meas std_msgs/Bool "data: true"
```
Don't send dummy measurements:
```
rostopic pub /sara_shield/send_dummy_meas std_msgs/Bool "data: false"
```
# Notes
* The password for the user `user` is `user`
* If something doesn't seem to be found, try re-sourcing 
```
cd tum_integration_ws/catkin_ws/
source devel/setup.bash
cd ..
source setup.bash
```

# Known issues
## Laptop with Nvidia Graphics Card (`libGL error`)
If you get an error
```
libGL error: MESA-LOADER: failed to retrieve device information
```
On laptops, you have to disable Nvidia's `on-demand` mode, which only turns on the GPU if needed.
To check your current mode, run
```
sudo prime-select query
```
If it says `on-demand`, switch to mode `nvidia` with 
```
sudo prime-select nvidia
```
**Restart your computer**
## Send no humans in scene
To tell sara-shield that there are no more humans in the scene, send
```
rostopic pub /sara_shield/humans_in_scene std_msgs/Bool "data: false" 
```

## Gravity compensation
In order to compansate gravity in simulation, do the following steps:
```
cd ~/tum_integration_ws/src/
git clone git@github.com:ADVRHumanoids/xbot2_examples.git
cd xbot2_examples
git checkout v2.10
```
then change following lines manually:
* line 40:
 remove `ControlMode::PosImpedance() + `
* line 187: remove `_robot->setPositionReference(_qref);`
then
```
cd ~/tum_integration_ws/build
mkdir xbot2_examples && cd xbot2_examples
cmake ../../src/xbot2_examples
ccmake ../../src/xbot2_examples
```
inside ccmake, change `/usr/lib/` to `/opt/xbot`
then
`sudo make -j4 install`

In the config, add
```
  gcomp_example:
      type: gcomp_example
      thread: rt_main
      params:
        enabled_chains: 
          value: [chain_E]
          type: vector<string>
```
  


