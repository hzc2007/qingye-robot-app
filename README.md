# 青野玄冬号 农业机器人控制 App 原型

这是一个可直接运行的 PWA / H5 App 原型，面向“模块化农业一体化机器人”：主机连接、地形规划、禁区设置、覆盖路径生成、自动驾驶任务下发、手动驾驶接管、急停、割草/除雪/集叶模块切换。

## 运行方式

1. 解压文件夹。
2. 双击 `index.html`，或用浏览器打开。
3. 手机端可用浏览器访问局域网地址，并“添加到主屏幕”，作为类 App 使用。
4. 实机连接时，将连接地址改成 Jetson Nano 提供的 WebSocket 服务，例如：

```text
ws://192.168.4.1:8765
```

## 推荐系统架构

```text
手机 App / 平板 App
   ↓ WebSocket / MQTT / HTTPS
Jetson Nano 上位计算平台
   ├─ 地图管理 / 轨迹规划 / 视觉避障 / RTK融合
   ├─ 自动驾驶状态机
   ↓ UART / CAN / RS485
STM32F407 实时控制器
   ├─ 左右履带电机闭环控制
   ├─ 割草/除雪/集叶模块控制
   ├─ 急停、安全互锁、电池保护
   ↓
电机驱动器 / 编码器 / BMS / 传感器
```

## 已实现功能

- 机器人连接页：支持 WebSocket 地址输入与连接状态显示。
- 首页：默认显示已连接，包含电量、速度、RTK状态、作业面积、模块状态、任务按钮、系统日志。
- 地图页：内置一张虚拟农场地图，包含工作区、禁区、通道、充电点、果树带、工具棚、水池等元素。
- 实时轨迹：机器人运动时用蓝色轨迹线显示已行驶路线，并统计轨迹长度、任务进度、机器人坐标。
- 路径规划：生成弓字形覆盖路径，模拟割草/除雪/集叶覆盖作业。
- 仿真自动驾驶：点击“开始自动作业”后，机器人会沿规划路径运动，可在地图页实时观察。
- 驾驶页：虚拟摇杆控制机器人线速度和角速度，顶部显示“实时速度”；“速度+ / 速度-”会同步改变首页速度卡片和设置页速度值。
- 安全控制：急停、急停复位、手动接管。
- 设置页：可手动修改首页显示的电量、速度和区域数值，便于比赛演示和参数调试。
- 地图导出：导出 `agripilot_virtual_map.json`。

## 对接机器人消息格式

App 发送给 Jetson Nano 的核心 JSON：

```json
{"type":"manual_drive","linear":0.3,"angular":0.1}
{"type":"map_upload","map":{}}
{"type":"task_start","module":"mower","map":{}}
{"type":"task_pause","pause":true}
{"type":"emergency_stop"}
{"type":"emergency_reset"}
{"type":"telemetry_override","battery":86,"speed":0.4,"area":965}
{"type":"manual_speed_adjust","speed":0.4,"speed_limit":0.4}
```

Jetson Nano 返回给 App 的核心 JSON：

```json
{"battery":86,"speed":0.4,"rtk":"RTK","pose":{"x":1.2,"y":3.4,"yaw":0.2}}
{"battery":86,"speed":0.4,"rtk":"RTK","pose":{"screen":true,"x":420,"y":280,"yaw":0.2}}
{"state":"AUTO_WORK","module":"mower","fault":null}
{"obstacle":{"x":2.1,"y":1.5,"level":"warning"}}
```

说明：`pose.screen=true` 表示 Jetson 直接回传画布坐标，适合比赛演示；不带 `screen` 时，App 按米制坐标和地图比例换算到虚拟地图中。

## 上线前必须补全

- 用户登录和设备绑定。
- TLS 加密与 Token 鉴权。
- 本地局域网模式和云端远程模式隔离。
- 急停硬件优先级高于 App 指令。
- 地图坐标从屏幕像素转换为 RTK / SLAM 世界坐标。
- A* / D* Lite / TEB / Pure Pursuit 等正式导航算法。
- 禁区、边界、坡度、低电量、丢定位、通信中断保护。

## 2026-06-26 更新：作业中实时调速

- 自动作业过程中允许点击「速度+」和「速度-」实时调整作业速度。
- 首页速度、手动驾驶页实时速度、设置页速度值会同步更新。
- 仿真机器人运动速度会跟随当前速度变化，不再固定为 0.4m/s。


## 实时探测地图
地图页新增 360° 激光雷达 / SLAM 风格实时探测窗口。在线演示时自动生成虚拟点云；实机可通过 WebSocket 下发 `lidar:[{angle,distance},...]` 数据，angle 单位为弧度，distance 单位为米。
