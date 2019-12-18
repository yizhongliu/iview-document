



# 马达控制

## 1   单个马达控制接口

### 接口

接口 1

```java
/**
     * 控制马达转动， 转动结束会通过 UnitySendMessage 通知unity
     * @param motorId: 马达id
    {
    @link HMotor 水平方向马达
    @link VMotor 垂直方向马达
    }
     * @param steps : PWM脉冲数量
     * @param dir : 转动方向
     * {
    @link    HMotorLeftDirection
    @link    HMotorRightDirection
    @link    VMotorUpDirection
    @link    VMotorDownDirection
     * }
     * @param delay 马达脉冲持续时间，单位us， 数值越小速度越快
     * @param bCheckLimitSwitch: 是否判断限位  true 判断， false 不判断
     */
    public int controlMotor(final int motorId, final int steps, final int dir, final int delay, final boolean bCheckLimitSwitch)
```





接口2

```java
  /**
     * 停止马达转动
     * @param motorId
     * {
     *     @link HMotor 水平方向马达
     *     @link VMotor 垂直方向马达
     * }
     */
    public void stopMotor(int motorId)
```



### 通知

接口1 数据反馈格式

```json
{
    "type":"callback",
    "action":"onMotorStop",
    "arg":{
        "motor":"horizotalMotor"
    }
}
```

| motorId | motor           |
| ------- | --------------- |
| HMotor  | horizontalMotor |
| VMotor  | verticalMotor   |



## 2  同时控制两个马达接口

### 接口

接口1

```java
    /**
     * 同时控制两个马达转动， 转动结束会通过 UnitySendMessage 通知unity
     * @param hDir : 水平马达转动方向
     * {
    @link    HMotorLeftDirection
    @link    HMotorRightDirection
     * }
     * @param hDelay: 水平马达脉冲持续时间， 单位us， 数值越小速度越快
     * @param vDir ： 垂直马达转动方向
     * {
    @link    VMotorUpDirection
    @link    VMotorDownDirection
     * }
     * @param vDelay 垂直马达脉冲持续时间，单位us， 数值越小速度越快
     * @param duration 马达转动时间， 单位ms
     * @param bCheckLimitSwitch: 是否判断限位  true 判断， false 不判断
     */
    public int controlMultiMotor(final int hDir, final int hDelay, final int vDir, final int vDelay, final int duration, final boolean bCheckLimitSwitch) 
```



接口2

```java
    /**
     * 同时控制两个马达转动， 参数选择不合适两个马达可能无法同时停止， 转动结束会通过 UnitySendMessage 通知unity
     * @param hSteps 水平马达转动的脉冲数
     * @param vSteps 垂直马达转动的脉冲数
     * @param hDir : 水平马达转动方向
     * {
    @link    HMotorLeftDirection
    @link    HMotorRightDirection
     * }
     * @param vDir ： 垂直马达转动方向
     * {
    @link    VMotorUpDirection
    @link    VMotorDownDirection
     * }
     * @param duration 马达运行时间，单位us
     * @param bCheckLimitSwitch: 是否判断限位  true 判断， false 不判断
     */
    public void controlMotors(final int hSteps, final int vSteps, final int hDir, final int vDir, final int duartion, final boolean bCheckLimitSwitch) 
```



### 通知

接口 数据反馈格式

```json
{
    "type":"callback",
    "action":"onMotorStop",
    "arg":{
        "motor":"multiMotor"
    }
}
```



## 3 以列表的方式控制马达

### 调用步骤

1 将马达控制命令添加到列表中

2 开始马达转动

### 接口

接口1 

```java
    /**
     * 添加马达转动指令
     * @param runningTime ：马达转动时间
     * @param hDirection ： 水平马达转动方向
     * @param hDelay ： 水平马达转动速度
     * @param vDirection ： 垂直马达转动方向
     * @param vDelay ： 垂直马达转动速度
     * @param bCheckLimitSwitch: 是否判断限位  true 判断， false 不判断
     */
    public void addMotorData(int runningTime, int hDirection, int hDelay, int vDirection, int vDelay, boolean bCheckLimitSwitch) 
```



接口2：

```java
    /**
     * 让马达按照设置的直接开始运行，每开始执行一条指令会通过接口通知unity
     */
    public void startMotorRunning()
```



### 通知

每开始执行一条指令通知的数据格式

```json
{
    "type":"callback",
    "action":"startMotorList",
    "arg":{
        "index":1
    }
}
```

所有指令执行完后通知的数据格式

```json
{
    "type":"callback",
    "action":"stopMotorList"
}
```



# 编码器接口

### 接口

接口1

```java
    /**
     * 启动监听编码其线程, 线程启动后会每隔50ms 通过UnitySendMessage 通知unity
     */
    public void startNotifyThread() 
```



接口2

```java
    /**
     * 停止监听编码其线程
     */
    public void stopNotifyThread()
```



接口3

```java
    /**
     * 获取编码器的值
     * @param encoderId 编码器id
     {
      @link HEncoder 水平方向编码器
      @link VEncoder 垂直方向编码器
     }
     * @return int 编码器脉冲数
     */
    public int getEncoder(int encoderId)
```



接口4

```java
    /**
     * 设置编码器的值
     * @param encoderId 编码器id
    {
    @link HEncoder 水平方向编码器
    @link VEncoder 垂直方向编码器
    }
     * @param value 编码器脉冲数
     */
    public void setEncoder(int encoderId, int value)
```



### 通知

接口1 通知的数据格式

```
{
    "type":"notify",
    "action":"getEncoder",
    "arg":{
        "horizontalDegree":1，
        "verticalDegree":0
    }
}
```



# Unity设置的回调接口

```java
    /**
     * 设置unity回调信息， android会通过 UnitySendMessage(String var0, String var1, String var2) 回调unity接口
     * @param unityPacakage : var0
     * @param unityMethod : var1
     */
    public void setUnityCallBack(String unityPacakage, String unityMethod)
```
