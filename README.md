# EGO-Planner 

An ESDF-free Gradient-based Local Planner for Quadrotors, modified from []()

**EGO-Planner** is a lightweight gradient-based local planner without ESDF construction, which significantly reduces computation time compared to some state-of-the-art methods <!--(EWOK and Fast-Planner)-->. The total planning time is only **around 1ms** and don't need to compute ESDF.

**Video Links:** [YouTube](https://youtu.be/UKoaGW7t7Dk), [bilibili](https://www.bilibili.com/video/BV1VC4y1t7F4/)


## Related Paper

EGO-Planner: An ESDF-free Gradient-based Local Planner for Quadrotors, Xin Zhou, Zhepei Wang, Chao Xu and Fei Gao (Accepted by RA-L). [arXiv Preprint](https://arxiv.org/abs/2008.08835), [IEEE Xplore](https://ieeexplore.ieee.org/abstract/document/9309347), and [IEEE Spectrum report](https://spectrum.ieee.org/automaton/robotics/robotics-hardware/video-friday-mit-media-lab-tf8-bionic-ankle).


## Acknowledgements
- The framework of this repository is based on [Fast-Planner](https://github.com/HKUST-Aerial-Robotics/Fast-Planner) by Zhou Boyu who achieves impressive proformance on quadrotor local planning.

- The L-BFGS solver we use is from [LBFGS-Lite](https://github.com/ZJU-FAST-Lab/LBFGS-Lite).
  It is a C++ head-only single file, which is lightweight and easy to use.

- The map generated in simulation is from [mockamap](https://github.com/HKUST-Aerial-Robotics/mockamap) by William Wu.

- The hardware architecture is based on an open source implemation from [Teach-Repeat-Replan](https://github.com/HKUST-Aerial-Robotics/Teach-Repeat-Replan).


## Release Note

- v1.2.1: update height of `end_pt_` to height of `final_goal`
- v1.2.0: support `POS_VEL_ACC` control
- v1.0.2: replace `prometheus_msgs` with `quadrotor_msgs`
- v1.0.1: mapping with D435i


## Compilation

```bash
catkin_make install --source src/EGO-Planner --build build/ego_planner
```

## Launch

```bash
roslaunch ego_planner simulation.launch
```

Then you can follow the gif below to control the drone.

<p align = "center">
<img src="pictures/sim_demo.gif" width = "640" height = "438" border="5" />
</p>


 ## Use GPU or Not
 Packages in this repo, **local_sensing** have GPU, CPU two different versions. By default, they are in CPU version for better compatibility. By changing
 
 ```
 set(ENABLE_CUDA false)
 ```
 
 in the _CMakeList.txt_ in **local_sensing** packages, to
 
 ```
 set(ENABLE_CUDA true)
 ```
 
CUDA will be turned-on to generate depth images as a real depth camera does. 

Please remember to also change the 'arch' and 'code' flags in the line of 
```
    set(CUDA_NVCC_FLAGS 
      -gencode arch=compute_61,code=sm_61;
    ) 
``` 
in _CMakeList.txt_. If you encounter compiling error due to different Nvidia graphics card you use or you can not see proper depth images as expected, you can check the right code via [link1](https://arnon.dk/matching-sm-architectures-arch-and-gencode-for-various-nvidia-cards/) or [link2](https://github.com/tpruvot/ccminer/wiki/Compatibility).
 
Don't forget to re-compile the code!

**local_sensing** is the simulated sensors. If ```ENABLE_CUDA``` **true**, it mimics the depth measured by stereo cameras and renders a depth image by GPU. If ```ENABLE_CUDA``` **false**, it will publish pointclouds with no ray-casting. Our local mapping module automatically selects whether depth images or pointclouds as its input.

For installation of CUDA, please go to [CUDA ToolKit](https://developer.nvidia.com/cuda-toolkit)

## Utilize the Full Performance of CPU

The computation time of our planner is too short for the OS to increase CPU frequency, which makes the computation time tend to be longer and unstable.

Therefore, we recommend you to manually set the CPU frequency to the maximum.
Firstly, install a tool by
```
sudo apt install cpufrequtils
```
Then you can set the CPU frequency to the maximum allowed by
```
sudo cpufreq-set -g performance
```
More information can be found in [http://www.thinkwiki.org/wiki/How_to_use_cpufrequtils](http://www.thinkwiki.org/wiki/How_to_use_cpufrequtils).

Note that CPU frequency may still decrease due to high temperature in high load.

## Improved ROS-RealSense Driver

We modified the ros-relasense driver to enable the laser emitter strobe every other frame, allowing the device to output high quality depth images with the help of emitter, and along with binocular images free from laser interference.

<p align = "center">
<img src="pictures/realsense.PNG" width = "640" height = "158" border="5" />
</p>

This ros-driver is modified from [https://github.com/IntelRealSense/realsense-ros](https://github.com/IntelRealSense/realsense-ros) and is compatible with librealsense2 2.30.0.
Tests are performed on Intel RealSense D435 and D435i.

Parameter ```emitter_on_off``` is to turn on/off the added function.
Note that if this function is turned on, the output frame rate from the device will be reduced to half of the frame rate you set, since the device uses half of the stream for depth estimation and the other half as binocular grayscale outputs.
What's more, parameters ```depth_fps``` and ```infra_fps``` must be identical, and ```enable_emitter``` must be true as well under this setting.

##  Install

The driver of librealsense2 2.30.0 should be installed explicitly.
On a x86 CPU, this can be performed easily within 5 minutes.
Firstly, remove the currently installed driver by 
```
sudo apt remove librealsense2-utils
```
or manually remove the files if you have installed the librealsense from source.
Then, you can install the library of version 2.30.0 by
```
sudo apt-key adv --keyserver keys.gnupg.net --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE || sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE
```

```
sudo add-apt-repository "deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo bionic main" -u
```
Then continue with
```
sudo apt-get install librealsense2-dkms
sudo apt install librealsense2=2.30.0-0~realsense0.1693
sudo apt install librealsense2-gl=2.30.0-0~realsense0.1693
sudo apt install librealsense2-utils=2.30.0-0~realsense0.1693
sudo apt install librealsense2-dev=2.30.0-0~realsense0.1693
sudo apt remove librealsense2-udev-rules
sudo apt install librealsense2-udev-rules=2.30.0-0~realsense0.1693
``` 
Here you can varify the installation by 
```
realsense-viewer
```

##  Run

If everything looks well, you can now compile the ros-realsense package named _modified_realsense2_camera.zip_ by ```catkin_make```, then run ros realsense node by 
```
roslaunch realsense_camera rs_camera.launch
```
Then you will receive depth stream along with binocular stream together at 30Hz by default.

<!--
# A Lightweight Quadrotor Simulator

The quadrotor simulator we use is inherited and modified from [Fast-Planner](https://github.com/HKUST-Aerial-Robotics/Fast-Planner). 
It is lightweight and super easy to use.
Only one topic is required to control the drone.
You can execute 
```
roslaunch so3_quadrotor_simulator simulator_example.launch 
```
to run a simple example in ego-planner/src/uav_simulator/so3/control/src/control_example.cpp.
If this simulator is helpful to you, plaease kindly give a star to [Fast-Planner](https://github.com/HKUST-Aerial-Robotics/Fast-Planner) as well.-->


## Maintaince
We are still working on extending the proposed system and improving code reliability. 

For any technical issues, please contact Xin Zhou (iszhouxin@zju.edu.cn) or Fei GAO (fgaoaa@zju.edu.cn).

For commercial inquiries, please contact Fei GAO (fgaoaa@zju.edu.cn).
