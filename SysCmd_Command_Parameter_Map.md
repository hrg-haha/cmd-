# SysCmd 命令常量与参数映射

说明：所有命令通过 NAME=<命令值> 进入 ExCmd 分发。表里的“参数示例”只展示业务参数写法，不重复展开整个 Dictionary。

## 1. 取证范围与漏项检查结果

本表只统计 `SysCommon.NtsCmd` 类下的 `M_*` 常量，并按下面规则做覆盖检查：

- 命令是否“已接入”，以 `Motion/NTS/BaseProcess.cs` 中 `BaseProcess.ExCmd(Dictionary<string, string> value)` 的 `switch` 为准。
- 命令的“标准方法”优先看 `BaseProcess`；如果 `Motion/NTS/机型接口处理/CT4A_Final_Process.cs` 有 `override`，则参数规则以 `CT4A_Final_Process` 为准。
- 只分析当前代码里显式读取的 `Dictionary<string, string> value` 键，不臆造隐式参数。

覆盖检查结果：

- `NtsCmd` 下 `M_*` 常量共 108 个。
- 其中 86 个已接入 `BaseProcess.ExCmd` 分发。
- 其中 12 个是通用参数键，不对应独立命令方法。
- 其中 10 个仅在 `SysCmd.cs` 中声明，当前未接入 `BaseProcess.ExCmd`。

额外说明：

- `M_CombineDarkPanel` 对应的方法名是 `M_COMBINEDARKPANEL`。
- `M_EndRun` 对应的方法名是 `M_ENDRUN`。
- `M_G4C_D_Control` 对应的方法名是 `M_G4C_Details_Control`。
- `M_G4C_Light_Control` 的 CT4A 实现里直接使用了字面量键 `"DATA"`，语义等同 `NtsCmd.M_Data`。
- `M_ICRWAVE` 直接使用了字面量键 `"ICR1"`、`"ICR2"`、`"ICR3"`、`"SleepTime"`，这些键当前没有在 `NtsCmd` 中定义常量。

## 2. BaseProcess 直接实现的已接入命令

| 功能/常量名 | 命令值 | 对应方法 | 参数示例 | 参数说明 |
| --- | --- | --- | --- | --- |
| M_AutoStart | M_AUTOSTART | BaseProcess.M_AutoStart | DATA=true | DATA: bool。设置机台自动启动状态。 |
| M_GetAutoStart | M_GETAUTOSTART | BaseProcess.M_GetAutoStart | 无 | 纯查询命令，返回当前 Auto 状态。 |
| M_AllHome | M_ALLHOME | BaseProcess.M_AllHome | 无 | 纯动作命令，直接触发整机回原。 |
| M_GetIoState | M_GetIOState | BaseProcess.M_GetIoState | 无 | 纯查询命令，返回当前 IO 状态快照。 |
| M_IsMotionReady | M_ISMOTIONREADY | BaseProcess.M_IsMotionReady | 无 | 纯查询命令，检查轴回零、急停、门、气源等状态。 |
| M_UnitInfo | M_UNITINFO | BaseProcess.M_UnitInfo | 无 | 当前实现没有读取业务参数。 |
| M_Socket_Enable | M_SOCKET_ENABLE | BaseProcess.M_Socket_Enable | DATA=0,true | A: DUT 编号。B: true/false。控制某个 DUT 的使能状态。 |
| M_Result | M_RESULT | BaseProcess.M_Result | DATA=pass<br>SOCKETID=0 | SOCKETID: 治具号。DATA: pass/fail/close。上报单个治具测试结果。 |
| M_Snd_Result | M_SND_RESULT | BaseProcess.M_Snd_Result | SOCKETID=0<br>DATA=短路报警 | SOCKETID: 治具号。DATA: 错误字符串。给指定治具写入错误描述。 |
| M_Alarm_App | M_ALARM_APP | BaseProcess.M_Alarm_App | DATA=机械手未到安全位 | DATA: 报警文本。发送给 APP。 |
| M_ChangeUnit | M_CHANGEUNIT | BaseProcess.M_ChangeUnit | DATA=3 | DATA: Index 编号。请求切换到指定 Unit。 |
| M_Retry | M_RETRY | BaseProcess.M_Retry | SOCKETID=0 | SOCKETID: 治具号。内部会换算 Index 并执行开夹具、真空检查、关夹具。 |
| M_GetTurnplateAngle | M_GETTURNPLATEANGLE | BaseProcess.M_GetTurnplateAngle | 无 | 纯查询命令，按当前转盘位置计算角度。 |
| M_Alarm | M_ALARM | BaseProcess.M_Alarm | DATA=GREEN<br>DATA=RED | DATA: GREEN/YELLOW/RED/BEEP/CLOSE。控制三色灯和蜂鸣器。 |
| M_SHOW_UI | M_SHOW_UI | BaseProcess.M_SHOW_UI | DATA=false | DATA: true/false。显示或隐藏 Motion 界面。 |
| M_RECYCLING | M_RECYCLING | BaseProcess.M_RECYCLING | DATA=true | DATA: true/false。设置是否清理产品标志。 |
| M_LIGHT | M_LIGHT | BaseProcess.M_LIGHT | DATA=SFRLIGHT,1,ON | A: 光源类型。B: 通道号。C: ON/OFF。按类型、通道和状态控制光源。 |
| M_ModuleSwitchControl | M_ModuleSwitchControl | BaseProcess.M_ModuleSwitchControl | SOCKETID=0<br>DATA=1,0 | SOCKETID: 起始治具号。A: 切换位1。B: 切换位2。控制模块切换两路输出。 |
| M_Module5VControl | M_Module5VControl | BaseProcess.M_Module5VControl | SOCKETID=0<br>DATA=1 | SOCKETID: 起始治具号。DATA: 0/1。控制模块 5V 电源。 |
| M_EndRun | M_ENDRUN | BaseProcess.M_ENDRUN | 无 | 停止测试收尾命令，是否关真空由配置决定。 |
| M_ICRCONTROL | M_ICRCONTROL | BaseProcess.M_ICRCONTROL | SOCKETID=1<br>DATA=1,0 | SOCKETID: 治具号。A: ICR 控制位1。B: ICR 控制位2。 |
| M_GHJDC54LIGHT | M_GHJDC54LIGHT | BaseProcess.M_GHJDC54LIGHT | DATA=1 | DATA: 通道号。1/2 对应不同输出，其它值表示全部关闭。 |
| M_SLEEP | M_SLEEP | BaseProcess.M_SLEEP（private） | DATA=500 | DATA: 延时毫秒数。线程休眠指定时长。 |
| M_ICRWAVE | M_ICRWAVE | BaseProcess.M_ICRWAVE（private） | SOCKETID=0<br>ICR1=0<br>ICR2=1<br>ICR3=2<br>SleepTime=200 | SOCKETID: 治具号。A: 第1段 ICR。B: 第2段 ICR。C: 第3段 ICR。D: 段间延时 ms。 |
| M_PLCMES | M_PLCMES | BaseProcess.M_PLCMES（private） | SOCKETID=0<br>DATA=1 | SOCKETID: 治具号。DATA: Fins 下为 int；Clownfish 下为字符串。用于 PLC/MES 停线或结果信息。 |
| M_PLCSPOTCHECK | M_PLCSPOTCHECK | BaseProcess.M_PLCSPOTCHECK（private） | SOCKETID=0<br>DATA=A1B1 | SOCKETID: 治具号。DATA: AABB 编码。A 段是点检信号，B 段是标样信号。 |

## 3. CT4A_Final_Process 覆盖实现的已接入命令

| 功能/常量名 | 命令值 | 对应方法 | 参数示例 | 参数说明 |
| --- | --- | --- | --- | --- |
| M_Index | M_INDEX | CT4A_Final_Process.M_Index | SOCKETID=0 | SOCKETID 可选。当前 CT4A 实现主要依赖机台配置和状态，SOCKETID 仅用于转盘耗时日志。 |
| M_Finish | M_FINISH | CT4A_Final_Process.M_Finish | STATION=STATION0 | STATION: 工位枚举字符串。根据工位决定退出黑白场、白场、中继镜、ALS、门等机构。 |
| M_GetIndexNum | M_GETINDEXNUM | CT4A_Final_Process.M_GetIndexNum | 无 | 纯查询命令，返回当前转盘 Index。 |
| M_LightPanel | M_LIGHTPANEL | CT4A_Final_Process.M_LightPanel | STATION=STATION0 | STATION: 工位枚举字符串。控制白场气缸进入测试位，并按工位做前置避让。 |
| M_DarkPanel | M_DARKPANEL | CT4A_Final_Process.M_DarkPanel | STATION=STATION0 | STATION: 工位枚举字符串。控制黑场气缸进入测试位，并按工位做前置避让。 |
| M_ExitDarkPanel | M_EXITDARKPANEL | CT4A_Final_Process.M_ExitDarkPanel | STATION=STATION0<br>DATA=0 | STATION: 工位。DATA: 0/1，可选。`0` 表示显式退出，缺省时走默认状态。 |
| M_DoorPanel | M_DOORPANEL | CT4A_Final_Process.M_DoorPanel | STATION=STATION0<br>SOCKETID=0<br>DATA=1 | STATION: 工位。SOCKETID: 治具号。DATA: 0/1。控制遮光门。 |
| M_MoveFocusPosition | M_MOVEFOCUSPOSITION | CT4A_Final_Process.M_MoveFocusPosition | 无 | 直接把机台移动到对焦测试位置。 |
| M_UV | M_UV | CT4A_Final_Process.M_UV | DATA=0,ON | A: 对象标识。B: ON/OFF。UV 控制命令。 |
| M_LIGHTCONTROL | M_LIGHTCONTROL | CT4A_Final_Process.M_LIGHTCONTROL | DATA=1<br>DATA=1,0\|2<br>STATION=STATION0 | A: 通道号。B: 控制器序号列表，可选。STATION 可选。用于 OTP16/多控制器光源控制。 |
| G4C 光源通道 M_G4C_Light_Control | M_G4C_Light_Control | CT4A_Final_Process.M_G4C_Light_Control | DATA=1 | DATA: 通道号。当前代码直接读字面量 `"DATA"`。 |
| G4C 明细控制 M_G4C_D_Control | M_G4CDCONTROL | CT4A_Final_Process.M_G4C_Details_Control | DATA=0,5,5,ON | A: 通道。B: 照度。C: 色温。D: ON/OFF。G4C 明细光源控制。 |
| AOI 光源 M_AOI_Light_Control | M_AOI_LIGHT_CONTROL | CT4A_Final_Process.M_AOI_Light_Control | DATA=1,ON | A: 通道。B: ON/OFF。AOI 光源控制。 |
| M_CHIP18_24CYLINDER | M_CHIP18_24CYLINDER | CT4A_Final_Process.M_CHIP18_24CYLINDER | STATION=STATION0<br>DATA=true | STATION: 工位。DATA: bool，可选。18 度灰卡/24 色卡切换；缺省时走配置默认值。 |
| 小白板 M_WHITEPANELBRIGHTCONTROL | M_WHITEPANELBRIGHTCONTROL | CT4A_Final_Process.M_WhitePanelBrightControl | STATION=STATION1<br>DATA=120 | STATION: 工位。DATA: 亮度值 ushort。设置小白板光源亮度。 |
| M_SETG5CCOLORTEMPILLUMINATION | M_SETG5CCOLORTEMPILLUMINATION | CT4A_Final_Process.M_SETG5CCOLORTEMPILLUMINATION | DATA=0,5,5000,255<br>DATA=1,6500,-<br>STATION=STATION0 | 格式1：光源ID,通道,色温,照度。格式2：通道,色温,照度。3 段格式时需要配合 STATION。 |
| M_FINAL4RELAY | M_FINAL4RELAY | CT4A_Final_Process.M_FINAL4RELAY | 无 | 直接把增距镜位移轴移动到测试位置。 |
| M_G4CLIGHTCONTROL | M_G4CLIGHTCONTROL | CT4A_Final_Process.M_G4CLIGHTCONTROL | DATA=1,on<br>DATA=2,off | A: 通道。B: on/off，可选。G4C 光源控制；未传 B 时默认按 on 处理。 |
| 千明光源 M_ThousliteLight | M_THOUSLITELIGHT | CT4A_Final_Process.M_ThousliteLight | DATA=2 | DATA: uint 模式/通道值。 |
| M_DUTPOWERCONTROL | M_DUTPOWERCONTROL | CT4A_Final_Process.M_DUTPOWERCONTROL | SOCKETID=0<br>DATA=true | SOCKETID: 治具号。DATA: 0/1/true/false。控制 DUT 供电开关。 |
| M_BEGINRUN | M_BEGINRUN | CT4A_Final_Process.M_BEGINRUN | 无 | 开始测试前做安全检查并置位运行状态。 |
| 组合黑场 M_CombineDarkPanel | M_COMBINEDARKPANEL | CT4A_Final_Process.M_COMBINEDARKPANEL | STATION=STATION0<br>DATA=0 | STATION: 工位。DATA: 0/1。组合黑场气缸动作。 |
| 转接板 M_ADPATERCONTROL | M_ADPATERCONTROL | CT4A_Final_Process.M_ADPATERCONTROL | SOCKETID=0<br>DATA=1 | SOCKETID: 治具号。DATA: 0/1/true/false。转接板总开关控制。 |
| 转接板 IO M_ADPATER_SwitchingIO | M_ADPATER_SWITCHINGIO | CT4A_Final_Process.M_ADPATER_SwitchingIO | SOCKETID=0<br>DATA=1,0 | SOCKETID: 治具号。A: 状态。B: IO 编号。转接板单路 IO 切换。 |
| M_RELAYMIRRORPANEL | M_RELAYMIRRORPANEL | CT4A_Final_Process.M_RELAYMIRRORPANEL | STATION=STATION0 | STATION: 工位枚举字符串。中继镜进入测试位。 |
| 电能表 M_PowerMonitor | M_POWERMONITOR | CT4A_Final_Process.M_PowerMonitor | DATA=C0 | DATA: 十六进制寄存器地址。电能表读取命令。 |
| M_ALSPANEL | M_ALSPANEL | CT4A_Final_Process.M_ALSPANEL | STATION=STATION1 | STATION: 工位枚举字符串。ALS 气缸进入测试位。 |
| 黑白场 M_BlackWhiteSwitching | M_BLACKWHITESWITCHING | CT4A_Final_Process.M_BlackWhiteSwitching | STATION=STATION1<br>DATA=0 | STATION: 工位。DATA: 0/1，可选。黑白场切换；`0` 常用于退出。 |
| M_Probe | M_PROBE | CT4A_Final_Process.M_Probe | SOCKETID=0<br>DATA=1 | SOCKETID: 治具号。DATA: 0/1/true/false。探针上下控制。 |
| M_OTPListLight | M_OTPLISTLIGHT | CT4A_Final_Process.M_OTPListLight | DATA=0,1,ON,100<br>DATA=0,1,ON,100,2 | A: 光源索引。B: 通道号。C: ON/OFF。D: 亮度。E: 光源组索引，可选。 |
| 治具水平 M_Socket_Horizontal | M_SOCKET_HORIZONTAL | CT4A_Final_Process.M_Socket_Horizontal | SOCKETID=0<br>DATA=true | SOCKETID: 治具号。DATA: 0/1/true/false。治具水平滑动控制。 |
| M_VacuumControl | M_VACUUMCONTROL | CT4A_Final_Process.M_VacuumControl | SOCKETID=0<br>DATA=ON | SOCKETID: 治具号。DATA: ON/OFF 或 0/1。指定治具真空控制。 |
| 系统 6 路 M_SystemSwitchControl | M_SYSTEMSWITCHCONTROL | CT4A_Final_Process.M_SystemSwitchControl | SOCKETID=0<br>DATA=1,0,1,0,1,0 | SOCKETID: 治具号。A-F: 6 路开关位。常见含义：DET、Diag、V_Mut、WE3.3V、OS、WE1.8V。 |
| 校正白场 M_CheckLightPANEL | M_CHECKLIGHTPANEL | CT4A_Final_Process.M_CheckLightPANEL | STATION=STATION0 | STATION: 工位枚举字符串。校正白场气缸进入测试位。 |
| M_GetCurrent | M_GETCURRENT | CT4A_Final_Process.M_GetCurrent | SOCKETID=0<br>ID=1555752920202<br>SN=DEVICESN<br>DATA=1 | SOCKETID: 治具号。ID: 业务记录 ID。SN: 产品 SN。DATA: 通道/模式值。用于获取当前产品电流。 |
| M_GetLineType | M_GETLINETYPE | CT4A_Final_Process.M_GetLineType | 无 | 纯查询命令，返回当前机台自动线/通讯类型。 |
| M_AlarmMES | M_ALARMMES | CT4A_Final_Process.M_AlarmMES | SOCKETID=0 | SOCKETID: 治具号。设置 PLC/MES 报警状态。 |
| M_Multimeter_O | M_MlULTIMETE_O | CT4A_Final_Process.M_Multimeter_O | DATA=1 | DATA: 通道号/测量模式值。万用表直接测阻值。 |
| 适配板万用表 M_Multimeter_ADPATER | M_MULTIMETE_ADPATER | CT4A_Final_Process.M_Multimeter_ADPATER | SOCKETID=0<br>DATA=2 | SOCKETID: 治具号。DATA: 1-3。打开适配板并测阻值。 |
| 德祐光源 M_DeYouLightList | M_DEYOULIGHTLIST | CT4A_Final_Process.M_DeYouLightList | DATA=0,1,ON,100 | A: 光源索引。B: 通道号。C: ON/OFF。D: 亮度。 |
| M_BarometerValue | M_BAROMETERVALUE | CT4A_Final_Process.M_BarometerValue | 无 | 纯查询命令，读取气压和温度。 |
| LED M_LEDPANEL | M_LEDPANEL | CT4A_Final_Process.M_LEDPANEL | STATION=STATION0 | STATION: 工位枚举字符串。LED 气缸进入测试位。 |
| 阻抗 M_IMPEDANCE_SWITCH | M_IMPEDANCE_SWITCH | CT4A_Final_Process.M_IMPEDANCE_SWITCH | SOCKETID=0<br>DATA=1,1,0 | SOCKETID: 治具号。A: switch。B: mode0。C: mode1。阻抗治具 IO 切换。 |
| M_GetImpedance | M_GETIMPEDANCE | CT4A_Final_Process.M_GetImpedance | SOCKETID=0<br>SN=DEVICESN | SOCKETID: 治具号。SN: 产品 SN。获取阻抗值。 |
| 轩士佳 M_SciencaCount | M_SCIENCACOUNT | CT4A_Final_Process.M_SciencaCount | DATA=0,2 | A: count。B: channel。轩士佳光源列表控制。 |
| ICR M_ICRSwitch | M_ICRSWITCH | CT4A_Final_Process.M_ICRSwitch | SOCKETID=0 | SOCKETID: 治具号。ICR 电流切换/读取相关控制。 |
| 萤烁光源 M_YSKJLightControl | M_YSKJLightControl | CT4A_Final_Process.M_YSKJLightControl | DATA=2<br>DATA=2,on | A: channel。B: status，可选。萤烁光源控制。 |
| M_MachineTemperature | M_MACNINETEMPERATURE | CT4A_Final_Process.M_MachineTemperature | STATION=STATION0 | STATION: 工位枚举字符串。获取机台温度。 |
| M_GetMachineVoltage | M_MACNINEVOLTAGE | CT4A_Final_Process.M_GetMachineVoltage | SOCKETID=0<br>STATION=STATION0 | SOCKETID: 治具号。STATION: 工位。获取机台电压。 |
| M_IsFinishHome | M_ISFINISHHOME | CT4A_Final_Process.M_IsFinishHome | 无 | 纯查询命令，返回机台是否完成复位。 |
| 翻转 M_Flip | M_FLIP | CT4A_Final_Process.M_Flip | SOCKETID=0<br>DATA=1 | SOCKETID: 治具号。DATA: 0/1/true/false。治具侧推翻转控制。 |
| M_SetPlcSN | M_SETPLCSN | CT4A_Final_Process.M_SetPlcSN | SOCKETID=0<br>DATA=SN12345 | SOCKETID: 治具号。DATA: SN 字符串。给 PLC 写入 SN。 |

## 4. 已接入分发但当前未实现的命令

这些常量已经在 `BaseProcess.ExCmd` 的 `switch` 里有分发，但当前代码里仍然是 `throw new NotImplementedException()`，并且 `CT4A_Final_Process` 也没有覆盖实现，因此无法从当前代码确认业务参数。

| 功能/常量名 | 命令值 | 对应方法 | 参数示例 | 参数说明 |
| --- | --- | --- | --- | --- |
| M_PrAndLaser | M_PRANDLASER | BaseProcess.M_PrAndLaser | 无 | 当前代码未实现。 |
| M_PreFocus | M_PREFOCUS | BaseProcess.M_PreFocus | 无 | 当前代码未实现。 |
| M_PrAndLaser_Dispense | M_PRANDLASER_DISPENSE | BaseProcess.M_PrAndLaser_Dispense | 无 | 当前代码未实现。 |
| M_Dispense | M_DISPENSE | BaseProcess.M_Dispense | 无 | 当前代码未实现。 |
| M_MoveTest | M_MOVETEST | BaseProcess.M_MoveTest | 无 | 当前代码未实现。 |
| M_MoveChart | M_MOVECHART | BaseProcess.M_MoveChart | 无 | 当前代码未实现。 |
| M_ADAPTER_SWITCH | M_ADAPTER_SWITCH | BaseProcess.M_ADAPTER_SWITCH | 无 | 当前代码未实现。 |
| M_WLIGHTPANEL | M_WLIGHTPANEL | BaseProcess.M_WLIGHTPANEL | 无 | 当前代码未实现。 |

## 5. 通用参数键（不对应独立命令方法）

| 功能/常量名 | 命令值 | 对应方法 | 参数示例 | 参数说明 |
| --- | --- | --- | --- | --- |
| M_Name | NAME | ExCmd 统一入口 | NAME=M_PLCMES | ExCmd 先靠它决定走哪个 case，所有命令都依赖。 |
| M_Data | DATA | 通用参数键 | DATA=true<br>DATA=1,0<br>DATA=pass | 最通用的业务参数载体，布尔、数值、复合字符串都可能放这里。 |
| M_Station | STATION | 通用参数键 | STATION=STATION0 | 用于把同一个命令路由到不同工位、气缸、测试位。 |
| M_Sn | SN | 通用参数键 | SN=DEVICESN | 产品序列号。 |
| M_Backlash | BACKLASH | 通用参数键 | BACKLASH=0.02 | 预留给运动参数类命令的回程间隙。 |
| M_Top | TOP | 通用参数键 | TOP=1 | 预留字段。 |
| M_Mode | MODE | 通用参数键 | MODE=ABS | 预留给模式切换命令。 |
| M_Id | ID | 通用参数键 | ID=1555752920202 | 业务记录 ID。 |
| M_Delay | DELAY | 通用参数键 | DELAY=200 | 预留延时参数，单位 ms。 |
| M_Interval | INTERVAL | 通用参数键 | INTERVAL=1000 | 预留间隔参数，单位 ms。 |
| M_Speed | SPEED | 通用参数键 | SPEED=50 | 预留速度参数。 |
| M_SocketId | SOCKETID | 通用参数键 | SOCKETID=0 | 指定具体治具、Socket 或与治具绑定的对象。 |

## 6. 仅在 SysCmd.cs 声明、当前未接入 BaseProcess.ExCmd 的常量

这些常量在 `SysCommon/NtsCmd` 里有声明，但当前没有被 `BaseProcess.ExCmd` 的 `switch` 接入，因此没有对应的方法入口。

| 功能/常量名 | 命令值 | 对应方法 | 参数示例 | 参数说明 |
| --- | --- | --- | --- | --- |
| M_MICROTEST | M_MICROTEST | 当前无对应 case | 无 | 仅声明，当前未接入命令分发。 |
| M_MIDDLETEST | M_MIDDLETEST | 当前无对应 case | 无 | 仅声明，当前未接入命令分发。 |
| M_FARTEST | M_FARTEST | 当前无对应 case | 无 | 仅声明，当前未接入命令分发。 |
| M_FOCUS4DARK | M_FOCUS4DARK | 当前无对应 case | 无 | 仅声明，当前未接入命令分发。 |
| M_FOCUS4LIGHT | M_FOCUS4LIGHT | 当前无对应 case | 无 | 仅声明，当前未接入命令分发。 |
| M_FOCUS4RELAY | M_FOCUS4RELAY | 当前无对应 case | 无 | 仅声明，当前未接入命令分发。 |
| M_MicroChartUp | M_MICROCHARTUP | 当前无对应 case | 无 | 仅声明，当前未接入命令分发。 |
| 振动 M_VIBRATE | M_VIBRATE | 当前无对应 case | 无 | OIS 振动开始命令，但当前未接入。 |
| M_VIBSTOP | M_VIBSTOP | 当前无对应 case | 无 | OIS 振动停止命令，但当前未接入。 |
| M_AUTOCHECK | M_AUTOCHECK | 当前无对应 case | 无 | 进入/退出点检模式，但当前未接入。 |

## 7. 使用建议

- 查“这个命令到底收哪些键”，先看本表的“参数示例”和“参数说明”。
- 查“这个命令实际跑的是 BaseProcess 还是 CT4A”，先看“对应方法”。
- 如果某条命令在本表里被标成“未实现”，说明当前代码只能确认常量已分发，但还不能确认业务参数格式。
- 如果某条命令带“字面量 DATA”或 `ICR1/ICR2/ICR3/SleepTime` 这类说明，表示它没有完全复用 `NtsCmd` 的通用参数常量，调用方要特别注意键名必须和当前代码一致。
