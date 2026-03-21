# CAN

## 一句话说明
CAN（Controller Area Network）是面向实时控制场景的多主机串行总线，依靠报文 ID 仲裁来保证高优先级消息优先发送，广泛用于机器人和汽车电子。

## 核心特征
- **多主通信**：总线上多个节点都可发起发送，无需中心主站。
- **差分传输**：`CAN_H/CAN_L` 抗共模干扰能力强，适合电机、电源噪声环境。
- **按 ID 仲裁**：ID 越小优先级越高，仲裁不破坏报文内容（无损仲裁）。
- **广播机制**：所有节点都能接收总线报文，由节点本地过滤 ID。
- **实时性可控**：通过合理分配 ID 优先级与总线负载可获得较稳定时延。

## 拓扑与布线要点
- **推荐拓扑**：总线型（线型主干 + 短支线），避免星型长分支。
- **终端电阻**：主干两端各 `120Ω`，中间节点不加终端。
- **支线长度**：越短越好（高速率下更严格），否则反射会导致误码。
- **地参考**：建议节点间共享参考地，降低共模漂移风险。
- **速率与长度折中**：速率越高，可用主干越短（工程上需联合实测）。

## CAN FD（Flexible Data-rate）
相较经典 CAN，CAN FD 的关键改进：
- **数据段更长**：单帧数据从 8 字节提升到最多 64 字节。
- **数据段可加速**：仲裁段保持传统速率，数据段可切换更高速率（BRS）。
- **效率更高**：在大数据量周期通信中可显著降低总线占用。

适用建议：
- 控制命令较短、兼容旧设备优先：可继续使用经典 CAN。
- 关节状态批量上报、参数块传输：优先考虑 CAN FD。

## Linux SocketCAN 实用命令
> 以下命令用于联调阶段快速验证链路与收发能力。

- 启动经典 CAN（示例 `can0`，1Mbps）：
  - `sudo ip link set can0 up type can bitrate 1000000`
- 启动 CAN FD（仲裁 1Mbps，数据段 2Mbps）：
  - `sudo ip link set can0 up type can bitrate 1000000 dbitrate 2000000 fd on`
- 关闭接口：
  - `sudo ip link set can0 down`
- 抓包查看：
  - `candump can0`
- 发送测试帧（经典 CAN）：
  - `cansend can0 123#1122334455667788`
- 发送测试帧（CAN FD，16 字节示例）：
  - `cansend can0 123##0112233445566778899AABBCCDDEEFF`

## OpenCan（openarm_can）对照
在本项目中，`openarm_can` 通过 `CANSocket` 封装 Linux SocketCAN，核心能力包括：
- 接口初始化：按网卡名（如 `can0`）创建并初始化 socket。
- 模式选择：通过 `enable_fd` 决定是否启用 CAN FD。
- 原始帧读写：`read_raw_frame()` / `write_raw_frame()`。
- 类型化读写：
  - 经典 CAN：`read_can_frame()` / `write_can_frame()`
  - CAN FD：`read_canfd_frame()` / `write_canfd_frame()`
- 非阻塞轮询：`is_data_available(timeout_us)` 用于低时延循环读取。

实践建议：
- 先在系统层用 `candump/cansend` 验证硬件与线束，再接入 ROS2 节点。
- 明确 ID 规划（控制、状态、诊断分区）并固化优先级规则。
- 对关键控制链路设置超时与重发策略，避免单点报文丢失影响动作安全。
