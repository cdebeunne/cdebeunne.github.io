# SaDVIO

This library provide a modular C++ framework dedicated on research about Visual Odometry (VO) and Visual Inertial Odometry (VIO). It also contains implementations of research works done at ISAE such as: factor graph sparsification, traversability estimation with 3D mesh and non overlapping field of view VO. This version is compatible with the middleware ROS2. 

<p align='center'>
    <img src="video.gif" alt="drawing" width="800"/>
</p>


# Installation

Here are the dependencies:
* [OpenCV](https://github.com/opencv/opencv/tree/4.10.0) for image processing
* [CERES](http://ceres-solver.org/installation.html) a non linear solver from google
* [Eigen](https://eigen.tuxfamily.org/dox/GettingStarted.html) a linera algebra library
* [yaml-cpp](https://github.com/jbeder/yaml-cpp) to parse config files 

You can either run this code with [ROS2](http://docs.ros.org/en/humble/Installation.html), the famous robotic middleware, or without ROS2. For now it has been tested with Galactic and Humble. A few changes are needed for Jazzy but only in the ros folder.

# Tutorials 

<a href="md__t_u_t_o.html"> Using SaDVIO with your own Dataset  </a>

<a href="md__t_u_t_o_l_i_b.html"> Using SaDVIO as an external library  </a>
