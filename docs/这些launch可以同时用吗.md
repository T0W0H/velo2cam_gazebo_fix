## 1. 这些 launch 能不能同时用？

**同一个 launch 的多份变体不能同时用，不同场景的 launch 也尽量不要同时用。**

原因是它们会互相冲突：

| 冲突点         | 说明                                          |
| ----------- | ------------------------------------------- |
| Gazebo 实例   | 每个 launch 都会启动 Gazebo，重复启动会报错               |
| 模型名         | 比如 `sensor2`、`target` 这些名字是固定的，重名会 spawn 失败 |
| 话题名         | `/velodyne_points`、图像话题等会重复                 |
| TF frame_id | 同名 frame 会互相覆盖                              |

### 哪些可以“凑着用”？

可以组合的是**功能互补**的 launch：

```bash
# 1. 启动 Gazebo 仿真
roslaunch velo2cam_gazebo mono_hdl64_p1_real.launch gui:=true

# 2. 同时另开一个终端，启动标定算法
roslaunch velo2cam_calibration velo2cam_calibration.launch

# 3. 再开一个终端启动 RViz
rosrun rviz rviz
```

这三个可以一起用，因为它们分管不同功能。

### 总结

| 情况 | 能不能同时用 |
|------|-------------|
| `mono_hdl64_p1_real.launch` 和 `mono_hdl64_p2_real.launch` | 不能，模型/话题冲突 |
| `single_pose/mono_hdl64/*.launch` 内部的 ideal/noise/real | 不能，三者只能选一个 |
| Gazebo launch + 标定 launch + RViz | 可以 |
| Gazebo launch + record_gazebo.launch | 可以（录制当前场景的话题） |

## 2. `.rviz` 文件建议放哪？

惯例是放在 ROS 包的 `rviz/` 目录下：

```
velo2cam_gazebo/
├── launch/
├── rviz/
│   └── calibration_view.rviz
└── package.xml
```

然后在 launch 文件里这样加载：

```xml
<node name="rviz" pkg="rviz" type="rviz"
      args="-d $(find velo2cam_gazebo)/rviz/calibration_view.rviz"/>
```

这样别人拿到包后，直接 `roslaunch` 就能加载你保存好的 RViz 配置。

### 如果你想新建

可以这样做：

```bash
mkdir -p /home/twh/文档/catkin_ws/src/velo2cam_gazebo/velo2cam_gazebo/rviz
```

然后在 RViz 里点 **File → Save Config As...**，保存到上面的 `rviz/` 目录里。

如果只是想临时看看，也可以直接保存到 `~/.rviz/` 下面，但那样就不属于这个 ROS 包了，别人用的时候找不到。