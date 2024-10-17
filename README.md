# SLAM with Jackal UGV Project

This project implements 2D SLAM and autonomous navigation for Clearpath Jackal UGVs using ROS Melodic on Ubuntu 18.04, in both Gazebo simulated and real environments. It covers navigation, gmapping, and localization, along with instructions for workspace setup and establishing connections between Jackal robots and remote devices.

Developed by Anton Trublin as part of the YorkU RAY program at York University, under the supervision of Prof. Costas Armenakis, PEng.

## Robot Platform

Two Jackal UGVs were utilized in this project, each with a distinct sensor configuration:

<table>
  <tr>
    <td><img src="figures/Jackal1.png" width="200">
    <p align="center">Jackal 1</p></td>
    <td>
      - Camera: Point Grey Flea3<br>
      - 2D LiDAR: SICK LMS111<br>
      - IMU: MicroStrain 3DM-GX5-25
    </td>
    <td><img src="figures/Jackal2.png" width="180">
    <p align="center">Jackal 2</p></td>
    <td>
      - Camera: Stereo Bumblebee BB2<br>
      - 3D LiDAR: Puck LITE Velodyne VLP-16<br>
      - IMU: MicroStrain 3DM-GX5-25
    </td>
  </tr>
</table>

## Demo Videos
Here are two example videos demonstrating the project capabilities:

#### SLAM (gmapping) with Gazebo
[Link to Jackal gmapping with Gazebo](https://drive.google.com/file/d/17Sv-ov9Swhs6AZY9-54BHi-h3nHwJ0zW/view)
<p align="center">
  <img src="figures/gazGmappingSpeedup.gif" width="888">
</p>

#### SLAM (gmapping) with Jackal 1
[Link to Jackal 1 gmapping in real environment](https://drive.google.com/file/d/1KalWldOSjY0e3EDhIbN7KeGoV3Dc38Ee/view)
<p align="center">
  <img src="figures/jackal1GmappingSpeedup.gif" width="888">
</p>

Additional demo videos can be found at the end of this README.

## Setup Instructions

#### Prerequisites
- Ubuntu 18.04
- ROS Melodic

Ensure that Ubuntu 18.04 and ROS Melodic are installed on your device. 
#### Workspace Setup
1. Install required ROS packages:
```
sudo apt-get update
sudo apt-get install ros-melodic-jackal-simulator ros-melodic-jackal-desktop ros-melodic-jackal-navigation
```
2. Create a workspace:
```
mkdir -p ~/jackal_1_workspace/jackal_navigation/src
cd ~/jackal_1_workspace/jackal_navigation/src
catkin_init_workspace
```
3. Clone this repository:
```
git clone https://github.com/trubyant/SLAM_with_JackalUGV.git
```
4. Copy pre-installed packages from the robot (if any):
```
scp -r [robot_username]@[robot_ip]:~/[catkin_ws_name]/src/* .
```
It will ask for the robot's password.

5. Install dependencies:
```
cd ~/jackal_1_workspace/jackal_navigation
rosdep install --from-paths src --ignore-src -r -y
```
6. Build the workspace:
```
catkin_make
```
7. Set up the connection between the robot and remote device. Create an executable file (e.g., remote-jackal-1.sh) with the following content:
```
export ROS_MASTER_URI=http://[robot_ip]:11311
export ROS_IP=[your_device_ip]
```
Make it executable:
```
chmod +x ~/jackal_1_workspace/remote-jackal-1.sh
```
Note: For seamless communication between the robot and your device, you may need to update the `/etc/hosts` file on both the robot and your device. However, if you're using IP addresses directly in the ROS_MASTER_URI and ROS_IP environment variables, this step may not be necessary.

SSH into the robot with its username and password to make sure that both sides can ping each other.
## Running SLAM

Note: Source the workspace and the connection script in each new terminal:
```
source ~/jackal_1_workspace/jackal_navigation/devel/setup.bash
source ~/jackal_1_workspace/remote-jackal-1.sh
```
#### In Gazebo Simulation
Note: Only source the workspace .bash file and not the executable .sh file when working with Gazebo.
1. Launch the simulation:
```
roslaunch jackal_gazebo custom_world.launch config:=front_laser
```
2. For gmapping (creating a map):
```
roslaunch jackal_navigation gmapping_demo.launch
```
3. For visualization:
```
roslaunch jackal_viz view_robot.launch config:=gmapping
```
#### With Real Jackal
1. For gmapping (creating a map):
```
roslaunch jackal_navigation gmapping_demo.launch scan_topic:=/scan
```
2. For visualization:
```
roslaunch jackal_viz view_robot.launch config:=gmapping
```
#### Saving the Map
Once you're satisfied with the map created using gmapping, you can save it:
1. Open a new terminal and source your workspace.
2. Navigate to the directory where you want to save the map:
```
cd ~/jackal_1_workspace/jackal_navigation/maps
```
3. Save the map:
```
rosrun map_server map_saver -f your_map_name
```
This will create two files: your_map_name.pgm (the map image) and your_map_name.yaml (the map metadata).
#### Additional Commands
Note: For real environment remember to add `scan_topic:=/scan` at the end of the command for launch file in `jackal_navigation` package.
1. For navigation without a map launch the following:
```
roslaunch jackal_navigation odom_navigation_demo.launch
```
for visualization:
```
roslaunch jackal_viz view_robot.launch config:=navigation
```
2. For localization (navigation with a pre-existing map):
```
roslaunch jackal_navigation amcl_demo.launch map_file:=/[full_path_to_map_file]/your_map_name.yaml
```
for visualization:
```
roslaunch jackal_viz view_robot.launch config:=localization
```
3. Resetting the robot position to the origin:

In RViz, if the robot is shifted from the origin odom frame (due to previous displacement), before 
launching any files, we can center the robot with the command below (applies for gmapping and localization).
```
roslaunch jackal_navigation reset_odometry.launch 
```
## Future Work

- Implement 3D SLAM with Jackal 2 utilising its 3D LiDAR and stereo camera
- Upgrade to the latest ROS2 version and the latest Ubuntu OS
- Utilize Jetson TX2 IMU for real-time processing capabilities

## Additional Demo Videos

#### Dynamic obstacle avoidance (localization) with Gazebo
[Link to Jackal localization with Gazebo](https://drive.google.com/file/d/1QzJ4Q-RuvKa2QtajLESE4XeJjScEocde/view)
<p align="center">
<img src="figures/gazLocalizationSpeedup.gif" width="888">
</p>

#### Dynamic obstacle avoidance (localization) with Jackal 1
[Link to Jackal 1 localization in real environment example 1](https://drive.google.com/file/d/1HEVnrSPohhcd4nkFsCFLTRPruPQncvNW/view)
<p align="center">
<img src="figures/jackal1LocalizationSpeedup1.gif" width="888">
</p>

[Link to Jackal 1 localization in real environment example 2](https://drive.google.com/file/d/1xy9nevqT50cjugIHf5Tl9ZtXu3YT_T9c/view)
<p align="center">
<img src="figures/jackal1LocalizationSpeedup2.gif" width="888">
</p>

#### SLAM (gmapping) with Jackal 2
[Link to Jackal 2 gmapping in real environment](https://drive.google.com/file/d/1AG-RoXqj5g46r8V6VE3--_nUpYgGnD_F/view)
<p align="center">
<img src="figures/jackal2GmappingSpeedup.gif" width="888">
</p>

## Acknowledgements

Special thanks to Prof. Costas Armenakis for his supervision and guidance during the YorkU RAY program.
