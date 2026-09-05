# 农业机器人 App 通信协议建议

## 1. 连接方式

开发调试阶段：WebSocket 局域网连接，App 与 Jetson Nano 在同一 Wi-Fi。
比赛展示阶段：WebSocket + 仿真数据，确保演示稳定。
实机应用阶段：MQTT/HTTPS + 局域网兜底，远程控制必须做鉴权与加密。

## 2. 主题/接口

### App → Robot

| 类型 | 说明 | 示例 |
|---|---|---|
| hello | 握手 | `{type:"hello", app:"AgriPilot"}` |
| manual_drive | 手动驾驶 | `{type:"manual_drive", linear:0.2, angular:0.1}` |
| map_upload | 下发地图 | `{type:"map_upload", map:{...}}` |
| task_start | 开始任务 | `{type:"task_start", module:"mower"}` |
| task_pause | 暂停任务 | `{type:"task_pause", pause:true}` |
| emergency_stop | 急停 | `{type:"emergency_stop"}` |
| emergency_reset | 急停复位 | `{type:"emergency_reset"}` |
| module_select | 模块切换 | `{type:"module_select", module:"leaf"}` |

### Robot → App

| 类型 | 说明 | 示例 |
|---|---|---|
| telemetry | 遥测 | `{battery:86, speed:0.4, rtk:"FIX"}` |
| pose | 位姿，App 会把位姿点加入实时轨迹 | `{pose:{x:1.2,y:3.4,yaw:0.2}}` 或 `{pose:{screen:true,x:420,y:280,yaw:0.2}}` |
| state | 状态 | `{state:"AUTO_WORK", module:"mower"}` |
| obstacle | 障碍物 | `{obstacle:{x:2.1,y:1.5,level:"warning"}}` |
| fault | 故障 | `{fault:{code:"LOW_BATTERY", message:"电量过低"}}` |

## 3. 自动驾驶状态机

```text
IDLE 待机
  ↓ 接收地图和任务
READY_READY 待启动
  ↓ 开始任务
AUTO_WORK 自动作业
  ├─ 障碍物 → AVOIDING 避障
  ├─ 低电量 → RETURN_HOME 返航
  ├─ 暂停 → PAUSED
  ├─ 通信中断 → SAFE_STOP
  └─ 急停 → EMERGENCY_STOP
```

## 4. 地图数据结构

```json
{
  "scale": 0.05,
  "area": [{"x":100,"y":120}],
  "ngz": [{"x":300,"y":220}],
  "pathway": [{"x":80,"y":500}],
  "robot": {"x":130,"y":500,"theta":0},
  "dock": {"x":106,"y":574},
  "coverage": [{"x":150,"y":160}],
  "trace": [{"x":130,"y":500}]
}
```

实际应用时，`x/y` 应转成米制坐标、经纬度或 ENU 坐标。App 当前版本也支持 `trace` 字段保存机器人实时轨迹，用于回放和作业覆盖率统计。

## 5. 安全要求

1. 急停必须硬件直连 STM32，不依赖手机 App。
2. App 失联超过 1~2 秒，机器人进入安全停车。
3. 刀盘、风机、除雪机构必须有独立使能和互锁。
4. 禁止未授权设备连接机器人。
5. 云端远控必须加密，并限制高危操作。

## 2026-06-26 更新：手动驾驶实时速度

手动驾驶页顶部显示实时速度，不再显示“限速”。点击 App 中的“速度+ / 速度-”时，首页速度、驾驶页实时速度和设置页速度值会同步变化。

```json
{"type":"manual_speed_adjust","speed":0.4,"speed_limit":0.4}
```

其中 `speed` 用于 App 界面实时显示，`speed_limit` 可作为 Jetson / STM32 侧的手动驾驶速度限制参考值。
