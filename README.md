# velo2cam_gazebo快速开始与理解

这个项目的**入口**就是它所有的 `.launch` 文件，核心都集中在：

```
velo2cam_gazebo/velo2cam_gazebo/launch/
```

## 1. 最常用的启动入口

README 里给的示例：

```bash
roslaunch velo2cam_gazebo mono_hdl64_p1_real.launch
```

对应文件：

```
velo2cam_gazebo/launch/journal/single_pose/mono_hdl64/mono_hdl64_p1_real.launch
```

这个 launch 会：

1. 启动 Gazebo 并加载 `worlds/calibration_scene.world`
2. 在 Gazebo 中生成：
   - 一个白色二维码标定板（`calibration_whiteqr_pattern`）
   - 一个 Blackfly 相机模型（`blackflys`）
   - 一个 Velodyne HDL-64 LiDAR 模型（`velodyne_HDL64`）
3. 发布 ROS 话题，例如：
   - `/velodyne_points`（点云）
   - `/stereo_camera/left/image_rect_color` 等图像话题

## 2. 其他 launch 入口

`launch/journal/` 下按实验配置分了很多子目录：

| 类别 | 含义 |
|------|------|
| `single_pose/` | 单一位姿场景 |
| `multi_pose/` | 多个位姿场景 |
| `single_sensor/` | 单个传感器 |
| `record/` | 录制 rosbag 用的 launch |

命名规律大致是：

```
mono_hdl64_p1_real.launch    # 单目相机 + HDL64 LiDAR, pose1, 真实噪声
mono_hdl64_p1_ideal.launch   # 同上，理想无噪声
mono_hdl64_p1_noise.launch   # 同上，噪声场景
```

## 3. 录制 bag 的入口

```bash
roslaunch velo2cam_gazebo record_gazebo.launch
```

它会启动 `rosbag record`，录制这些话题：

- `/stereo_camera/left/camera_info`
- `/stereo_camera/left/image_rect_color`
- `/stereo_camera/right/camera_info`
- `/stereo_camera/right/image_rect_color`
- `/velodyne_points`
- `/tf`、`/tf_static`

## 4. 实际做标定时还会启动什么？

这个项目（`velo2cam_gazebo`）只负责**仿真产生数据**。

真正做 LiDAR-camera 外参标定，还要启动另一个仓库里的节点，通常是：

```bash
roslaunch velo2cam_calibration velo2cam_calibration.launch
```

所以完整流程一般是：

1. 启动 Gazebo 仿真：

```bash
roslaunch velo2cam_gazebo mono_hdl64_p1_real.launch gui:=true
```

2. 启动标定算法：

```bash
roslaunch velo2cam_calibration velo2cam_calibration.launch
```

3. （可选）用 RViz 可视化点云和图像，Fixed Frame 设为点云的 `frame_id` 即可。

可以按 **"入口 → 组合 → 变体"** 三层来理解。

## 1. 先看顶层入口

所有 launch 都在这个目录：

```
velo2cam_gazebo/velo2cam_gazebo/launch/
├── record_gazebo.launch          # 录制 rosbag
└── journal/                      # 论文实验用的仿真场景
    ├── multi_pose/               # 多个位姿
    ├── record/                   # 录制专用
    ├── single_pose/              # 单一位姿
    └── single_sensor/            # 单个传感器
```

## 2. `journal/` 里的命名规律

每个子目录里的 launch 文件名都长这样：

```
<传感器组合>_p<位姿编号>_<噪声类型>.launch
```

例如：

```
mono_hdl64_p1_real.launch
│     │      │  │
│     │      │  └── 噪声类型：ideal(无噪声) / noise(有噪声) / real(真实)
│     │      └── 位姿编号：p1, p2, p3...
│     └── 传感器组合
└── 场景分类：single_pose / multi_pose / single_sensor
```

### 传感器组合

| 组合                          | 含义                 |
| --------------------------- | ------------------ |
| `mono`                      | 单目相机               |
| `stereo`                    | 双目相机               |
| `hdl32` / `hdl64` / `vlp16` | Velodyne LiDAR 型号  |
| `mono_hdl64`                | 单目相机 + HDL64 LiDAR |
| `stereo_hdl32`              | 双目相机 + HDL32 LiDAR |
| `hdl32_hdl64`               | 两个 LiDAR           |

## 3. 一般怎么选？

| 你想做什么    | 启动哪个                                                        |
| -------- | ----------------------------------------------------------- |
| 只想跑个示例看看 | `mono_hdl64_p1_real.launch`                                 |
| 做标定实验    | 同上，或者 `single_pose` 下的对应组合                                  |
| 研究多组位姿影响 | `multi_pose` 下的                                             |
| 只验证单个传感器 | `single_sensor` 下的                                          |
| 录数据包     | `record_gazebo.launch` 或 `journal/record/record_xxx.launch` |

## 4. 最常用的一条

```bash
roslaunch velo2cam_gazebo mono_hdl64_p1_real.launch gui:=true
```

如果只想快速跑通，就记这一条就行。




### Q&A:

[这个项目的入口是哪些?我会启动哪些?](docs/这个项目的入口是哪些?我会启动哪些?.md)

[好多launch文件,眼花缭乱](docs/好多launch文件,眼花缭乱.md)

[这些launch可以同时用吗](docs/这些launch可以同时用吗.md)

[这些launch有固定的项目吗](docs/这些launch有固定的项目吗.md)

[那我可以通过开启多个launch来组合吗](docs/那我可以通过开启多个launch来组合吗.md)

[录制是啥](docs/录制是啥.md)

[录制怎么用](docs/录制怎么用.md)