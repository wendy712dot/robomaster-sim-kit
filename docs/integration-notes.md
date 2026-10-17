# 来源、改动与验证记录

记录日期：2026-10-18。

## 来源与固定基线

- 第一套自瞄方案：<https://github.com/wendy712dot/simplified-auto-aim-system>
- 仿真器仓库：<https://github.com/wendy712dot/bevy_robomaster_simulator>
- 选定分支：`fix/ros2-interface-compatibility`
- 固定基线提交：`7c075703d99192b940d1310bf674253f5d08d532`
- 第二套方案（仅借鉴接口/看门狗经验）：<https://github.com/lijjinyu-coder/armor_aim/tree/feature/simulator-integration>
- 上游来源：<https://github.com/HNUYueLuRM/bevy_robomaster_simulator>、<https://github.com/Blackjack200/bevy_robomaster_simulator>

本仓库不修改上述原仓库，不合并两套视觉算法。第二套方案的 `/rm_gimbal/cmd`、PID 和 TF 控制方式不属于本仓库所选仿真器协议。

## 本仓库整理内容

- 保留固定提交的仿真器源码、ROS2 消息包、锁文件和完整运行资产。
- ROS2-only 构建路径包含 `BevyDefault` 导入兼容处理。
- 默认关闭 `power_rune.auto_aim_enabled` 和 `vision_test_pose`，从通用出生点和手动控制开始。
- 增加相对路径的环境、构建和运行脚本。
- 增加图像/内参探针、有限时长云台探针，以及默认禁用控制的误差适配器。
- 增加适配器角度换算单元测试。
- 保留第一套 C++ 自瞄源码作为可选例程，不携带其大视频和输出文件。
- 补充根 README、接口说明和本记录；不发布背景 PDF 或个人路径。

## 本机实测结果

环境：Ubuntu 22.04、ROS2 Humble、Rust 1.95.0，2026-10-18。

| 检查 | 结果 |
|---|---|
| 草稿完整性 | 186 个文件、约 97 MiB；复制后逐文件 SHA-256 一致；模型资产齐全 |
| 行尾/路径 | 未发现 CRLF；未发现发布代码中的个人绝对路径 |
| Shell 语法 | `env.sh`、`build.sh`、`run.sh` 通过 `bash -n` |
| Python 单元测试 | 7/7 通过 |
| ROS2 消息包 | `rm_interfaces` 由 colcon 构建成功 |
| 仿真器 release 构建 | `--locked --no-default-features --features ros2` 成功；首次约 6 分钟 |
| 可选 C++ ROS2 例程 | `auto_aim_ros2` 构建成功；有一条成员初始化顺序警告 |
| 仿真器启动 | 未通过：当前受限任务进程不能连接已有 X11 会话，Bevy 报 `XOpenDisplayFailed` |
| 图像/内参话题 | 未实测：仿真器未进入运行态 |
| 云台正负号、绝对角与停止 | 未实测：仿真器未进入运行态 |
| 闭环视觉效果 | 未实测；不采用来源仓库的历史结论代替 |

Rust 构建另有 4 条非阻塞警告（两个未使用 re-export、`BevyDefault` 未使用及弃用）。由于构建成功且运行阻塞发生在 X11 连接阶段，本次没有据此改动仿真逻辑。

## 仍需在可访问图形/GPU 的会话完成

1. `vulkaninfo --summary`，再运行 `bash scripts/run.sh`。
2. 核对节点、话题、`/camera_info`、实际 `rgb8` 图像尺寸和接收 FPS。
3. 以 `+5° yaw` 和小幅 pitch 命令确认正负号、绝对角语义及反馈。
4. 确认停止消息会清除追踪；确认强制退出时 `F5` 可接管。
5. 可选：在开环确认颜色和内参后运行 C++ 视觉例程，再启用闭环。

这些项目完成前，不能宣称图像、云台或闭环已经在本机通过。
