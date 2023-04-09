# 麦克纳姆轮小车开发记录
```
开发人员：杨工

工作记录：
————2023年4月9号
1、麦克纳姆轮小车模型建立，添加雷达、摄像头、陀螺仪数据
2、自定义地图数据
3、添加键盘遥控节点，WASD控制，QE为转向，Shift加速，其他键盘任意键暂停
4、gmapping建图，保存地图
5、amcl定位，move_base导航
6、cartographer建图、保存地图

```

**仿真测试**
仿真环境启动：
```
$ roslaunch atom atom_world.launch
```
键盘遥控：
```
$ rosrun atom teleop_cmd_vel
```

**gmapping建图**
启动建图：
```
$ roslaunch atom gmapping.launch
```

遥控小车走完建图区域
```
$ rosrun atom teleop_cmd_vel
```

保存地图：
```
$ rosrun map_server map_saver -f map
```

**amcl定位和move_base导航**
启动定位和导航：
```
$ roslaunch atom navigation.launch
```


**cartographer建图**
仿真环境启动：
```
$ roslaunch atom atom_world.launch
```

启动cartographer：
```
$ roslaunch cartographer_ros demo_revo_lds.launch
```

修改的东西：
cartographer_ros/cartographer_ros/launch/demo_revo_lds.launch
```
<launch>
  <param name="/use_sim_time" value="true" />

  <node name="cartographer_node" pkg="cartographer_ros"
      type="cartographer_node" args="
          -configuration_directory $(find cartographer_ros)/configuration_files
          -configuration_basename revo_lds.lua"
      output="screen">
    <remap from="scan" to="scan" />
  </node>

  <node name="rviz" pkg="rviz" type="rviz" required="true"
      args="-d $(find cartographer_ros)/configuration_files/demo_2d.rviz" />
</launch>
```
cartographer_ros/cartographer_ros/configuration_files/revo_lds.lua
```
include "map_builder.lua"
include "trajectory_builder.lua"

options = {
  map_builder = MAP_BUILDER,
  trajectory_builder = TRAJECTORY_BUILDER,
  map_frame = "map",
  tracking_frame = "hokuyo_link",
  published_frame = "hokuyo_link",
  odom_frame = "odom",
  provide_odom_frame = false,
  publish_frame_projected_to_2d = false,
  use_odometry = false,
  use_nav_sat = false,
  use_landmarks = false,
  num_laser_scans = 1,
  num_multi_echo_laser_scans = 0,
  num_subdivisions_per_laser_scan = 1,
  num_point_clouds = 0,
  lookup_transform_timeout_sec = 0.2,
  submap_publish_period_sec = 0.3,
  pose_publish_period_sec = 5e-3,
  trajectory_publish_period_sec = 30e-3,
  rangefinder_sampling_ratio = 1.,
  odometry_sampling_ratio = 1.,
  fixed_frame_pose_sampling_ratio = 1.,
  imu_sampling_ratio = 1.,
  landmarks_sampling_ratio = 1.,
}

MAP_BUILDER.use_trajectory_builder_2d = true

TRAJECTORY_BUILDER_2D.submaps.num_range_data = 35
TRAJECTORY_BUILDER_2D.min_range = 0.2
TRAJECTORY_BUILDER_2D.max_range = 20.
TRAJECTORY_BUILDER_2D.missing_data_ray_length = 1.
TRAJECTORY_BUILDER_2D.use_imu_data = false
TRAJECTORY_BUILDER_2D.use_online_correlative_scan_matching = true
TRAJECTORY_BUILDER_2D.real_time_correlative_scan_matcher.linear_search_window = 0.1
TRAJECTORY_BUILDER_2D.real_time_correlative_scan_matcher.translation_delta_cost_weight = 10.
TRAJECTORY_BUILDER_2D.real_time_correlative_scan_matcher.rotation_delta_cost_weight = 1e-1

POSE_GRAPH.optimization_problem.huber_scale = 1e2
POSE_GRAPH.optimize_every_n_nodes = 35
POSE_GRAPH.constraint_builder.min_score = 0.65

return options
```

保存地图：
```
$ rosservice call /finish_trajectory 0
status: 
  code: 0
  message: "Finished trajectory 0."
```

```
$ rosservice call /write_state  "filename: '/home/ubuntu/map.pbstream'
include_unfinished_submaps: false" 
status: 
  code: 0
  message: "State written to '/home/ubuntu/map.pbstream'."

```
转化地图：
```
$ rosrun cartographer_ros cartographer_pbstream_to_ros_map  -map_filestem=/home/ubuntu/map -pbstream_filename=/home/ubuntu/map.pbstream -resolution=0.05
```








