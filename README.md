# RoboMaster Sim Kit

一个面向自瞄程序接入的轻量 RoboMaster 仿真仓库。仓库以 Bevy 仿真器为主体，保留 ROS2 消息、模型资产和最小探针；你可以直接发布仿真器原生控制消息，也可以参考小型适配器。这里不规定视觉算法或工程框架。

## 包含什么

- `simulator/`：固定基线的 Bevy 仿真器、资产和 `rm_interfaces`
- `examples/ros2_tools/`：图像探针、云台命令探针和可选误差适配器
- `examples/simplified_auto_aim/`：可选的 C++ 视觉例程，不是使用本仿真器的前置条件
- `docs/interfaces.md`：话题、单位、坐标和停止语义
- `docs/integration-notes.md`：来源、改动、本次验证与已知限制

## 环境

目标环境为 Ubuntu 22.04、ROS2 Humble、支持 Vulkan 的图形会话。需要 Rust 1.95.0（由 `simulator/rust-toolchain.toml` 固定）、C/C++ 工具链、Clang/libclang，以及 Bevy 所需的 X11/Wayland、Vulkan 和音频开发库。

先按 ROS 官方说明安装 Humble，再安装常用依赖：

```bash
sudo apt update
sudo apt install -y git curl build-essential cmake pkg-config clang libclang-dev \
  libssl-dev libudev-dev libasound2-dev libx11-dev libxi-dev libxrandr-dev \
  libxcursor-dev libxkbcommon-dev libwayland-dev libvulkan-dev \
  mesa-vulkan-drivers vulkan-tools python3-colcon-common-extensions \
  ros-humble-ros-base ros-humble-geometry-msgs ros-humble-sensor-msgs \
  ros-humble-tf2-msgs ros-humble-visualization-msgs ros-humble-nav-msgs \
  ros-humble-rosidl-default-generators
rustup toolchain install 1.95.0 --profile minimal
```

构建可选视觉例程还需要 OpenCV 与 cv_bridge：

```bash
sudo apt install -y libopencv-dev python3-opencv ros-humble-cv-bridge \
  ros-humble-ament-index-cpp
```

## 构建与启动

在仓库根目录运行：

```bash
bash scripts/build.sh
bash scripts/run.sh
```

首次 Rust release 构建耗时较长。仿真窗口出现后，按 `F5` 开启自动云台控制；默认配置故意保持关闭，避免未知发布者一启动就接管云台。

另开终端检查图像和内参：

```bash
source scripts/env.sh
python3 examples/ros2_tools/image_probe.py
```

可选显示图像（需要 OpenCV/cv_bridge）：

```bash
python3 examples/ros2_tools/image_probe.py --ros-args -p show:=true
```

在确认只有一个控制发布者、并已按 `F5` 后，用小角度测试绝对角命令：

```bash
python3 examples/ros2_tools/gimbal_probe.py --yaw 5 --pitch 0 --seconds 2
python3 examples/ros2_tools/gimbal_probe.py --stop
```

接自己的算法时，最直接的方式是订阅 `/image_raw` 与 `/camera_info`，读取 `/vision_receive_data`，完成坐标和单位转换后发布 `/vision_send_data`。无目标、结果过期或程序退出前必须主动发布无效目标；仅停止发布不会清除仿真器保存的目标。详见 [接口说明](docs/interfaces.md)。

## 可选例程

```bash
bash scripts/build.sh examples
source scripts/env.sh
ros2 run auto_aim_ros2 auto_aim_open_loop
```

闭环前先做开环观察；需要控制时再增加 `--ros-args -p enable_control:=true`。例程未附带原项目的大视频文件，参数也只适合特定颜色与场景，请勿把它当作通用视觉基准。

## 许可证与来源

仿真器部分沿用其 AGPL-3.0 许可证；可选视觉例程保留 MIT 许可证。精确来源、基线提交和本仓库改动见 [整合记录](docs/integration-notes.md)。
