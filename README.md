# MOLA SLAM & Localization Guide

This guide explains how to:

-   Run SLAM with LiDAR + IMU
    
-   Save and visualize maps
    
-   Perform localization using a prebuilt map
    
-   Generate occupancy grid maps for Nav2
    

----------

# 1. SLAM

## 1.1 SLAM with 3D LiDAR + IMU

    export  MOLA_TF_FOOTPRINT_LINK=''  
      
    ros2 launch mola_lidar_odometry ros2-lidar-odometry.launch.py \  
     lidar_topic_name:=/scan/points \  
     generate_simplemap:=True \  
     use_rviz:=False \  
     use_mola_gui:=False \  
     imu_topic_name:=/imu/data

----------

## 1.2 SLAM with 2D LiDAR + IMU

    export  MOLA_TF_FOOTPRINT_LINK=''  
      
    ros2 launch mola_lidar_odometry ros2-lidar-odometry.launch.py \  
     lidar_topic_name:=/scan \  
     lidar_topic_type:=LaserScan \  
     mola_lo_pipeline:=../pipelines/lidar2d.yaml \  
     imu_topic_name:=/imu/data \  
     generate_simplemap:=True \  
     use_rviz:=False \  
     use_mola_gui:=False

----------

## 1.3 Save Map

    ros2 service call /map_save mola_msgs/srv/MapSave \  
    "{map_path: '/tmp/my_map'}"

----------

## 1.4 Visualize Map

    mm-viewer -l libmola_metric_maps.so /path/to/your/map.mm

----------

# 2. Localization

## 2.1 Launch in Localization Mode

    export  MOLA_TF_FOOTPRINT_LINK=''  
      
    ros2 launch mola_lidar_odometry ros2-lidar-odometry.launch.py \  
     lidar_topic_name:=/scan \  
     lidar_topic_type:=LaserScan \  
     mola_lo_pipeline:=../pipelines/lidar2d.yaml \  
     imu_topic_name:=/imu/data \  
     generate_simplemap:=True \  
     use_rviz:=False \  
     use_mola_gui:=False \  
     start_active:=False \  
     start_mapping_enabled:=False

----------

## 2.2 Load Map

    ros2 service call /map_load mola_msgs/srv/MapLoad \  
    "{map_path: '/tmp/my_map'}"

----------

## 2.3 Activate Localization

    ros2 service call /mola_runtime_param_set mola_msgs/srv/MolaRuntimeParamSet \  
    "{parameters: \"mola::LidarOdometry:lidar_odom:\n  active: true\n\"}"

----------

# 3. Generate Occupancy Grid Map (for Nav2)

## 3.1 Convert `.simplemap` → `.mm`

    sm2mm \  
      -i /path/to/your/my_map.simplemap \  
      -o /path/to/your/my_map.mm \  
      -p /path/to/your/mola_ws/mp2p_icp/demos/sm2mm_bonxai_voxelmap_gridmap_no_deskew.yaml

**Note:**  
Customize the YAML configuration file for your setup.

----------

## 3.2 Convert `.mm` → Occupancy Grid

    mm2grid /path/to/your/my_map.mm \  
      --layer gridmap \  
      --occupied-thresh  0.80 \  
      --free-thresh  0.2

**Note:**  
You may need to post-process the generated occupancy grid map (e.g., using OpenCV).  
Refer to `map_filter_example.ipynb` for guidance.
