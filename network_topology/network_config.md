1. 设备在局域网中通信。
- 时序图
    ```mermaid
    sequenceDiagram
        participant DeviceA as 设备A<br/>(192.168.1.10)
        participant Switch as 交换机
        participant DeviceB as 设备B<br/>(192.168.1.20)
        
        Note over DeviceA,DeviceB: 场景：设备A向设备B发送数据
        
        DeviceA->>DeviceA: 检查目标IP是否在同一子网<br/>(192.168.1.0/24)
        DeviceA->>DeviceA: ARP请求：查询192.168.1.20的MAC地址
        DeviceA->>Switch: 广播ARP请求
        Switch->>DeviceB: 转发ARP请求到所有端口
        DeviceB->>DeviceB: 识别到目标IP是自己
        DeviceB->>Switch: ARP响应（包含MAC地址）
        Switch->>DeviceA: 转发ARP响应
        DeviceA->>DeviceA: 缓存MAC地址到ARP表
        
        Note over DeviceA,DeviceB: 开始数据传输
        
        DeviceA->>Switch: 发送数据帧（目标MAC: DeviceB）
        Switch->>Switch: 查找MAC地址表
        Switch->>DeviceB: 单播转发数据帧到DeviceB
        DeviceB->>DeviceB: 接收并处理数据
        DeviceB->>Switch: 发送ACK确认
        Switch->>DeviceA: 转发ACK到DeviceA
        DeviceA->>DeviceA: 确认传输成功
    ```
- 设备在局域网内通信在链路层会解析
2. 设备在互联网中通信。

3. 设备多网卡共存时的通信。

4. 常用术语解析：NAT、网关、交换机、路由器