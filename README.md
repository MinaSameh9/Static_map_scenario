# Static Map Scenario Setup Instructions

This document outlines the steps to set up and run a ROS (Robot Operating System) environment for a static map scenario using a pre-built map. Ensure all commands are executed in separate terminals, and set the `ROS_IP` environment variable in each terminal.

## Prerequisites
- Set the `ROS_IP` environment variable in **all terminals**:
  ```bash
  export ROS_IP=192.168.1.28  # Replace with your network IP
  ```

## Steps to Create and Save a Static Map

1. **Start the ROS Master**
   ```bash
   roscore
   ```

2. **Connect the Robot Serial Port (RSP) with ROS**
   ```bash
   rosrun rosserial_python serial_node.py tcp
   ```

3. **Publish the `motor_speed` Topic**
   ```bash
   rosrun control motor_controller.py
   ```

4. **Launch Freenect for Depth Camera**
   ```bash
   roslaunch freenect_launch freenect.launch depth_registration:=true
   ```

5. **Publish Static Transform between `camera_link` and `base_link`**
   ```bash
   rosrun tf2_ros static_transform_publisher 0 0 0 0 0 0 1 base_link camera_link
   ```

6. **Run RTAB-Map for SLAM with Visual Odometry**
   ```bash
   roslaunch rtabmap_ros rtabmap.launch rtabmap_args:="--delete_db_on_start" rtabmapviz:=false rviz:=true frame_id:=base_link subscribe_odom:=false visual_odometry:=true odom_topic:=/rtabmap/odom database_path:=~/catkin_ws/src/my_robot/maps/my_static_map.db
   ```

7. **View and Verify the Map Database**
   ```bash
   /opt/ros/noetic/bin/rtabmap-databaseViewer ~/catkin_ws/src/my_robot/maps/workspace_map/workspace_map.db
   ```

8. **Launch RTAB-Map with the Saved Database**
   ```bash
   roslaunch rtabmap_ros rtabmap.launch database_path:=~/catkin_ws/src/my_robot/maps/workspace_map/workspace_map.db use_gui:=false
   ```

9. **Save the Map as a Grid Map**
   ```bash
   rosrun map_server map_saver -f ~/catkin_ws/src/my_robot/maps/workspace_map/workspace_map map:=/rtabmap/grid_map
   ```

## Steps to Use the Static Map with TEB Navigation

1.**Launch Map Server with the Saved Map**
   ```bash
   rosrun map_server map_server ~/catkin_ws/src/maps/my_static_map.yaml
   ```
2. **Launch RTAB-Map in Localization Mode**
   ```bash
   roslaunch rtabmap_ros rtabmap.launch localization:=true rtabmapviz:=false rviz:=true frame_id:=base_link
   ```

3. **Launch TEB Navigation**
   ```bash
   roslaunch static_scenario static_nav.launch
   ```

## Additional Instructions
- Use the **2D Pose Estimate** tool in RViz to update the robot's position relative to the scanned map.
- Ensure the paths to the map files (`~/catkin_ws/src/my_robot/maps/my_static_map.db`, `~/catkin_ws/src/static_scenario/maps/my_static_map.yaml`) are correct and accessible.
- Verify that all required ROS packages (`rosserial_python`, `freenect_launch`, `rtabmap_ros`, `tf2_ros`, `map_server`, `control`, and `static_scenario`) are installed and configured properly.

## Notes
- Replace `192.168.1.28` with your actual network IP.
- Each command should be run in a separate terminal with `ROS_IP` set.
- The map database and YAML files should be consistent across the steps to ensure proper localization and navigation.
