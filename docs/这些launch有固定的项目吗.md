## 1. 所有 launch 都有的“固定项目”

是的，每个 launch 都会做这些相同的事：

| 固定项目          | 说明                                              |
| ------------- | ----------------------------------------------- |
| **启动 Gazebo** | 通过 `empty_world.launch` 启动                      |
| **加载 world**  | `worlds/calibration_scene.world`                |
| **生成标定板**     | 模型名 `target`，来自 `calibration_whiteqr_pattern`   |
| **生成至少一个传感器** | 相机、LiDAR 或两者都有                                  |
| **发布 ROS 话题** | 图像 `/stereo_camera/...`、点云 `/velodyne_points` 等 |

所以无论启动哪个 launch，**标定板 `target` 总是会存在的**，只是传感器的类型和数量不同。

## 2. 传感器的排列组合

这些 launch 基本覆盖了论文里常见的传感器配置，但**不是所有数学上的排列组合**。

### 当前覆盖的组合

#### `single_sensor/` — 单个传感器

| 目录        | 含义                    |
| --------- | --------------------- |
| `mono/`   | 单目相机                  |
| `stereo/` | 双目相机                  |
| `hdl32/`  | Velodyne HDL-32 LiDAR |
| `hdl64/`  | Velodyne HDL-64 LiDAR |
| `vlp16/`  | Velodyne VLP-16 LiDAR |

#### `single_pose/` — 两个传感器 + 一个位姿

| 目录              | 组合              |
| --------------- | --------------- |
| `mono_hdl64/`   | 单目相机 + HDL-64   |
| `mono_mono/`    | 单目相机 + 单目相机     |
| `stereo_hdl32/` | 双目相机 + HDL-32   |
| `stereo_hdl64/` | 双目相机 + HDL-64   |
| `hdl32_hdl64/`  | HDL-32 + HDL-64 |

#### `multi_pose/` — 多个位姿下的标定

| 目录                   | 组合                        |
| -------------------- | ------------------------- |
| `mono_hdl64/`        | 单目 + HDL-64               |
| `matlab_mono_hdl64/` | 用于 Matlab 工具箱的单目 + HDL-64 |
| `mono_mono/`         | 单目 + 单目                   |
| `stereo_hdl32/`      | 双目 + HDL-32               |
| `hdl32_hdl64/`       | HDL-32 + HDL-64           |

## 3. 是所有排列组合吗？

**不是严格的排列组合**，而是按论文实验需求挑选的常用组合。

比如下面这些组合就**没有**提供：

| 未提供的组合         | 可能原因             |
| -------------- | ---------------- |
| `mono_hdl32`   | 不是论文重点           |
| `mono_vlp16`   | 不是论文重点           |
| `stereo_hdl64` | 实际上有             |
| `stereo_vlp16` | 未覆盖              |
| `hdl64_hdl64`  | 同型号 LiDAR 标定意义不大 |
| 三传感器组合         | 太复杂，未覆盖          |

## 4. 怎么判断某个组合有没有？

可以直接搜：

```bash
find . -name "*vlp16*.launch"     # 看 VLP-16 相关的
find . -name "*mono_hdl32*.launch" # 看 mono+hdl32 有没有
```

## 5. 如果你想自己组合

看一个 launch 文件的结构，基本都是：

1. 加载 world
2. spawn `target`
3. spawn 第一个 sensor
4. spawn 第二个 sensor（如果有）

你可以复制一个最相近的 launch，改一下模型路径和参数，就能生成新的组合。比如以 `mono_hdl64_p1_real.launch` 为模板，把 HDL-64 换成 VLP-16。

---

所以结论是：**标定板固定存在，传感器是按实验需求组合好的，但没有穷举所有可能。**