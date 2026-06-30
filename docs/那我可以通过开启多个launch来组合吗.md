## 不行，不建议通过“开多个 launch”来组合场景

这些 launch **不是设计来组合用的**。每个 launch 本身就是一个完整场景。

### 为什么不能同时开？

每个 launch 内部都做了同样的事：

1. 启动 Gazebo
2. 加载同一个 world
3. 生成名为 `target` 的标定板
4. 生成名为 `sensors`、`sensor2` 等的模型
5. 发布 `/velodyne_points`、图像等话题

假设你同时开：

```bash
roslaunch velo2cam_gazebo mono_hdl64_p1_real.launch
roslaunch velo2cam_gazebo mono_mono_p1_real.launch
```

会冲突：

| 冲突                    | 结果                       |
| --------------------- | ------------------------ |
| 都启动了 Gazebo           | 第二个 Gazebo 起不来           |
| 都生成了 `target` 模型      | 第二个 `target` 重名，spawn 失败 |
| 都用 `/velodyne_points` | 两个话题混在一起                 |
| 都用 `sensor2` 等模型名     | 重名报错                     |

结果不是组合，是冲突和报错。

所以这样是拼不出新场景的。

---

## 正确的“组合”方式

### 1. 一个 launch 里 spawn 多个模型

如果你想自定义组合，应该**改 launch 文件**，把不同模型放到同一个 launch 里 spawn。

例如，你想 `mono + hdl64 + vlp16` 的组合，但项目里没有这个 launch，那你就复制一个最接近的 launch，把三个模型都写进去：

```xml
<!-- 生成标定板 -->
<param name="target" command="$(find xacro)/xacro ..." />
<node name="spawn_target_model" ... />

<!-- 生成相机 -->
<param name="sensors" command="$(find xacro)/xacro ..." />
<node name="spawn_sensor_model" ... />

<!-- 生成 HDL64 -->
<param name="sensor2" command="$(find xacro)/xacro ..." />
<node name="spawn_sensor_model2" ... />

<!-- 生成 VLP16 -->
<param name="sensor3" command="$(find xacro)/xacro ..." />
<node name="spawn_sensor_model3" ... />
```


### 2. 功能互补的 launch 可以一起开

只有**功能互补、不生成重叠资源**的 launch 才能同时开，比如：

```bash
# 终端 1：启动仿真
roslaunch velo2cam_gazebo mono_hdl64_p1_real.launch

# 终端 2：启动标定算法
roslaunch velo2cam_calibration velo2cam_calibration.launch

# 终端 3：启动可视化
rosrun rviz rviz
```

**或者**

```bash
# 1. 仿真
roslaunch velo2cam_gazebo mono_hdl64_p1_real.launch

# 2. 录制 bag
roslaunch velo2cam_gazebo record_gazebo.launch
```

这个可以，因为 `record_gazebo.launch` 只启动 `rosbag record`，不生成模型，不启动 Gazebo。

---

## 总结

| 你想做什么               | 做法          |
| ------------------- | ----------- |
| 同时开两个场景 launch      | ❌ 不行，会冲突    |
| 拼出新的传感器组合           | 改 launch 文件 |
| 仿真 + 录制 / 仿真 + 标定算法 | ✅ 可以，功能互补   |
