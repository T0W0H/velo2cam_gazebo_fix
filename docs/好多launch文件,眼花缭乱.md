是的，看着很多，但其实有规律。可以按 **"入口 → 组合 → 变体"** 三层来理解。

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