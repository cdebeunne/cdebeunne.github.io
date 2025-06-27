# Use SaDVIO as a Library

Here is a small tutorial that shows how to use SaDVIO as an external C++ library, and eventually to run a VO/VIO thread in your code pipeline. A nice example can be found in the following [code](https://github.com/cdebeunne/pg_fusion)

## Installation

First of all, you need to compile SaDVIO source code the [/cpp](https://github.com/ISAE-PNX/SaDVIO/tree/main/cpp) folder as such
```
mkdir build
cd build
cmake ..
sudo make install
```
This will ensure that all the header are well installed on your machine

## CMAKELISTS

In your `CMakeLists.txt` you can simply request the package with this line:
```
find_package(isae_slam REQUIRED)
```
And target link the library with the following tag:
```
target_link_libraries(${PROJECT_NAME} ${ISAE_SLAM_LIBRARIES} ...)
```

## Handle a VO/VIO thread

To do so you simply need to load the config file and launch a thread from the [SLAMCore](https://github.com/ISAE-PNX/SaDVIO/blob/main/cpp/include/isaeslam/slamCore.h) class. An example is provided in the following code snippet
~~~~~~~~~~~~~{.cpp}
#include "isaeslam/slamParameters.h"
#include <yaml-cpp/yaml.h>
#include <iostream>

int main(int argc, char **argv) {
    rclcpp::init(argc, argv);

    // Create the SLAM parameter object
    std::string path                                 = "path/to/your/config/folder";
    std::shared_ptr<isae::SLAMParameters> slam_param = std::make_shared<isae::SLAMParameters>(path);

    std::shared_ptr<isae::SLAMCore> SLAM;

    // Load the proper SLAMCore depending on your config
    if (slam_param->_config.slam_mode == "bimono")
        SLAM = std::make_shared<isae::SLAMBiMono>(slam_param);
    else if (slam_param->_config.slam_mode == "mono")
        SLAM = std::make_shared<isae::SLAMMono>(slam_param);
    else if (slam_param->_config.slam_mode == "nofov")
        SLAM = std::make_shared<isae::SLAMNonOverlappingFov>(slam_param);
    else if (slam_param->_config.slam_mode == "bimonovio")
        SLAM = std::make_shared<isae::SLAMBiMonoVIO>(slam_param);
    else if (slam_param->_config.slam_mode == "monovio")
        SLAM = std::make_shared<isae::SLAMMonoVIO>(slam_param);

    // Launch SLAM thread
    std::thread odom_thread(&isae::SLAMCore::runFullOdom, SLAM);
    odom_thread.detach();

    // Do things
    // That you want
    // To do

    return 0;
}
~~~~~~~~~~~~~

Once you have a thread that runs, you may develop your own SensorSubscriber to gather data, format them into frame and send them into the slam with the data provider:
~~~~~~~~~~~~~{.cpp}
std::shared_ptr<isae::Frame> frame = getFrameFromData(data);
SLAM->_slam_param->getDataProvider()->addFrameToTheQueue(frame);
~~~~~~~~~~~~~
Then, you can request the frames processed by the SLAM using the temporary variable `_frame_to_display` as such:
~~~~~~~~~~~~~{.cpp}
// Get the ouput from the SLAM
std::shared_ptr<isae::Frame> frame_ready  = _slam->_frame_to_display;;
while (frame_ready == nullptr) {
    frame_ready = _slam->_frame_to_display;
    std::this_thread::sleep_for(std::chrono::milliseconds(1));
}

// Get the pose of the frame (for instance)
Eigen::Affine3d T_w_f = frame_ready->getFrame2WorldTransform();
~~~~~~~~~~~~~
A more complete example is given in the [pipeline.cpp](https://github.com/cdebeunne/pg_fusion/blob/main/pipeline.cpp) source file of the pose graph fusion library.