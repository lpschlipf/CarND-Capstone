Project: System Integration(Capstone Project)
---


### Project overview

In this project, provide ROS nodes to for core functionality of an autonomous vehicle system which are traffic light detection, steering and throttle control and waypoint following. The structure and implementation of the code closely follows the [classroom descriptions](https://classroom.udacity.com/nanodegrees/nd013/parts/3205f941-2b69-42f8-8b82-6e1ad394fa84/modules/2495068d-1953-4fad-9e39-e7ad697b4d1b/lessons/28d758ea-1ef0-4cc6-a9d2-f058ef8b265c/concepts/50d790a3-23a5-404b-acb3-4b51640e8af9) and [this repository](https://github.com/snandasena/CarND-Capstone-System-Integration).

To install, please use **one** of the two installation options, either native **or** docker installation.

### Native Installation

* Be sure that your workstation is running Ubuntu 16.04 Xenial Xerus or Ubuntu 14.04 Trusty Tahir. [Ubuntu downloads can be found here](https://www.ubuntu.com/download/desktop).
* If using a Virtual Machine to install Ubuntu, use the following configuration as minimum:
  * 2 CPU
  * 2 GB system memory
  * 25 GB of free hard drive space

  The Udacity provided virtual machine has ROS and Dataspeed DBW already installed, so you can skip the next two steps if you are using this.

* Follow these instructions to install ROS
  * [ROS Kinetic](http://wiki.ros.org/kinetic/Installation/Ubuntu) if you have Ubuntu 16.04.
  * [ROS Indigo](http://wiki.ros.org/indigo/Installation/Ubuntu) if you have Ubuntu 14.04.
* Download the [Udacity Simulator](https://github.com/udacity/CarND-Capstone/releases).

### Docker Installation
[Install Docker](https://docs.docker.com/engine/installation/)

Build the docker container
```bash
docker build . -t capstone
```

Run the docker file
```bash
docker run -p 4567:4567 -v $PWD:/capstone -v /tmp/log:/root/.ros/ --rm -it capstone
```

### Port Forwarding
To set up port forwarding, please refer to the "uWebSocketIO Starter Guide" found in the classroom (see Extended Kalman Filter Project lesson).

### Usage

1. Clone the project repository
```bash
git clone https://github.com/udacity/CarND-Capstone.git
```

2. Install python dependencies
```bash
cd CarND-Capstone
pip install -r requirements.txt
```
3. Make and run styx
```bash
cd ros
catkin_make
source devel/setup.sh
roslaunch launch/styx.launch
```
4. Run the simulator

### Real world testing
1. Download [training bag](https://s3-us-west-1.amazonaws.com/udacity-selfdrivingcar/traffic_light_bag_file.zip) that was recorded on the Udacity self-driving car.
2. Unzip the file
```bash
unzip traffic_light_bag_file.zip
```
3. Play the bag file
```bash
rosbag play -l traffic_light_bag_file/traffic_light_training.bag
```
4. Launch your project in site mode
```bash
cd CarND-Capstone/ros
roslaunch launch/site.launch
```
5. Confirm that traffic light detection works on real life images

### Other library/driver information
Outside of `requirements.txt`, here is information on other driver/library versions used in the simulator and Carla:

Specific to these libraries, the simulator grader and Carla use the following:

|        | Simulator | Carla  |
| :-----------: |:-------------:| :-----:|
| Nvidia driver | 384.130 | 384.130 |
| CUDA | 8.0.61 | 8.0.61 |
| cuDNN | 6.0.21 | 6.0.21 |
| TensorRT | N/A | N/A |
| OpenCV | 3.2.0-dev | 2.4.8 |
| OpenMP | N/A | N/A |

We are working on a fix to line up the OpenCV versions between the two.


[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

#### System Architecture
As described in the classroom, the architecture of the project is structured as shown into ROS nodes:
<img src="imgs/final-project-ros-graph-v2.png" width="1000" height="600" />

#### Implementation structure

###### tl_detector/
This folder contains everything related to traffic light detection, mainly `tl_detector.py`. This node takes data from the `/image_color`, `/current_pose`, and `/base_waypoints` topics and publishes the locations when to stop on a red traffic light to the `/traffic_waypoint` topic.

The `/current_pose` provides the vehicle's **current position**, and `/base_waypoints` provides a complete list of **waypoints** the car will be following.

<img src="imgs/tl-detector-ros-graph.png" width="700" height="180" />

###### (path_to_project_repo)/ros/src/waypoint_updater/
This package contains the waypoint updater node: `waypoint_updater.py`. The purpose of this node is to update the target velocity property of each waypoint based on traffic light and obstacle detection data. This node will subscribe to the `/base_waypoints`, `/current_pose`, `/obstacle_waypoint`, and `/traffic_waypoint` topics, and publish a list of waypoints ahead of the car with target velocities to the `/final_waypoints` topic.

<img src="imgs/waypoint-updater-ros-graph.png" width="700" height="180" />

###### (path_to_project_repo)/ros/src/twist_controller/

Carla is equipped with a drive-by-wire (dbw) system, meaning the throttle, brake, and steering have electronic control. This package contains the files that are responsible for control of the vehicle: the node `dbw_node.py` and the file `twist_controller.py`, along with a pid and lowpass filter that you can use in our implementation. The dbw_node subscribes to the `/current_velocity` topic along with the `/twist_cmd` topic to receive target linear and angular velocities. 
Additionally, this node will subscribe to `/vehicle/dbw_enabled`, which indicates if the car is under dbw or driver control. This node will publish **throttle**, **brake**, and **steering** commands to the `/vehicle/throttle_cmd`, `/vehicle/brake_cmd`, and `/vehicle/steering_cmd` topics.

<img src="imgs/dbw-node-ros-graph.png" width="700" height="180" />


###### (path_to_project_repo)/ros/src/styx/
A package that contains a server for communicating with the simulator, and a bridge to translate and publish simulator messages to ROS topics.

###### (path_to_project_repo)/ros/src/styx_msgs/
A package which includes definitions of the custom ROS message types used in the project.

###### (path_to_project_repo)/ros/src/waypoint_loader/
A package which loads the static waypoint data and publishes to `/base_waypoints`.

###### (path_to_project_repo)/ros/src/waypoint_follower/
A package containing code from [Autoware](https://github.com/Autoware-AI/autoware.ai) which subscribes to `/final_waypoints` and publishes target vehicle linear and angular velocities in the form of twist commands to the `/twist_cmd` topic.

