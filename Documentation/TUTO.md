# Using SaDVIO with your own Dataset

This tutorial explains how to use the SaDVIO library with your own dataset. It covers the necessary steps to prepare your data, create a proper calibration file, and run the library.
Introduction

To use SaDVIO with your own dataset, you need to satsify the following:

* A dataset containing synchronized image measurements (+ IMU)
* A calibration file describing the intrinsic and extrinsic parameters of your sensor suite.

## Dataset Format

The configuration of your dataset should be stored in .yaml file, itself stored in a [folder](https://github.com/ISAE-PNX/SaDVIO/tree/main/ros/config) containing a dataset folder and a `config.yaml` file.
The config.yaml file contains all the SLAM related parameters, and the dataset folder should contain a bunch of .yaml file that describes each dataset.
For instance, the eth.yaml file (e.g. for the EUROC dataset) looks like:

```
# General sensor definitions.
dataset: EUROC
ncam: 2

camera_0:
  topic: /cam0/image_raw
  # Sensor extrinsics wrt. the body-frame.
  T_BS:
    cols: 4
    rows: 4
    data: [0.0148655429818, -0.999880929698, 0.00414029679422, -0.0216401454975,
		  0.999557249008, 0.0149672133247, 0.025715529948, -0.064676986768,
	    -0.0257744366974, 0.00375618835797, 0.999660727178, 0.00981073058949,
		  0.0, 0.0, 0.0, 1.0]

  # Camera specific definitions.
  rate_hz: 20
  resolution: [752, 480]
  projection_model: pinhole
  intrinsics: [458.654, 457.296, 367.215, 248.375] #fu, fv, cu, cv
  distortion_model: radial-tangential
  distortion_coefficients: [-0.28340811, 0.07395907, 0.00019359, 1.76187114e-05]

camera_1:
  topic: /cam1/image_raw
  # Sensor extrinsics wrt. the body-frame.
  T_BS:
    cols: 4
    rows: 4
    data: [0.0125552670891, -0.999755099723, 0.0182237714554, -0.0198435579556,
		  0.999598781151, 0.0130119051815, 0.0251588363115, 0.0453689425024,
	  -0.0253898008918, 0.0179005838253, 0.999517347078, 0.00786212447038,
		  0.0, 0.0, 0.0, 1.0]

  # Camera specific definitions.
  rate_hz: 20
  resolution: [752, 480]
  projection_model: pinhole
  intrinsics: [457.587, 456.134, 379.999, 255.238] #fu, fv, cu, cv
  distortion_model: radial-tangential
  distortion_coefficients: [-0.28368365,  0.07451284, -0.00010473, -3.55590700e-05]

imu:
  topic: /imu0
  T_BS:
    cols: 4
    rows: 4
    data: [1.0, 0.0, 0.0, 0.0,
          0.0, 1.0, 0.0, 0.0,
          0.0, 0.0, 1.0, 0.0,
          0.0, 0.0, 0.0, 1.0]
  rate_hz: 200

  # inertial sensor noise model parameters (static)
  gyroscope_noise_density: 1.6968e-04     # [ rad / s / sqrt(Hz) ]   ( gyro "white noise" )
  gyroscope_random_walk: 1.9393e-05       # [ rad / s^2 / sqrt(Hz) ] ( gyro bias diffusion )
  accelerometer_noise_density: 2.0000e-2  # [ m / s^2 / sqrt(Hz) ]   ( accel "white noise" )
  accelerometer_random_walk: 3.0000e-1    # [ m / s^3 / sqrt(Hz) ]  ( accel bias diffusion )
  dt_imu_cam: 0.0                         # Temporal offset between IMU and camera in seconds
``` 

So here you need to know precisely the transform between every sensor, as well as their intrinsic properties. A wonderfull toolbox that you can use for such a task is [kalibr](https://github.com/ethz-asl/kalibr) and [allan variance for ros](https://github.com/ori-drs/allan_variance_ros). Apart from that, a few parameters need some explanations:
* SaDVIO supports several camera models such as double sphere or omni camera model, you can find some example in the dataset folder
* The transform of the IMU should be the identity so that it is identifies as the base of the robot. This allow for smaller lever arms between sensor and a better stability of the pipeline.

Once you have properly edited your dataset, you can simply set the parameter `dataset_id` with the name of your yaml file (e.g. eth for eth.yaml) and run the program!
 
## Run Without ROS2

If you don't use the middleware ROS2, you can use the EUROC dataset ASL data format for which a data grabber is already implemented in ADataprovider.h. Then you can compile and run the executable [main.cpp](https://github.com/ISAE-PNX/SaDVIO/blob/main/cpp/main.cpp) in the [/cpp](https://github.com/ISAE-PNX/SaDVIO/tree/main/cpp) folder as such
```
mkdir build
cd build
cmake ..
make
```

And to run the executable
```
./isaeslam "/your/path/SaDVIO/ros/config" "/your/path/to/your/dataset/in/ASL/format"
```

## Run with ROS2

With ROS2, we have implemented all the interface with ros in the [/ros](https://github.com/ISAE-PNX/SaDVIO/tree/main/ros) folder.

You can compile the node with the following command:
```
colcon build --symlink-install --packages-select isae_slam_ros
```

We have set a nice launch file that runs the program with a cool rviz visualizer:
```
ros2 launch isae_slam_ros isae_slam.xml
```


