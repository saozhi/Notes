# 代码SVN

路径：

https://192.168.10.31:8443/svn/Robot_Dev/robot_task

usr_name:  liwenzhi

passcode:  my4X1ru8

# 底盘

## 登陆底盘

```shell
## ssh登陆到底盘, 用户名都是benzhang, 192.168.1.xx,密码1-6

liwenzhi@liwenzhi:~$ ssh benzhang@192.168.1.56

The authenticity of host '192.168.1.56 (192.168.1.56)' can't be established.
ECDSA key fingerprint is SHA256:98ytuYE7jFwpQiwwevAqGgcEi7FyAYehnafU81kNRfg.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.56' (ECDSA) to the list of known hosts.
benzhang@192.168.1.56's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.159 aarch64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Fri May 15 15:08:56 2020 from 192.168.1.44

benzhang@localhost:~$ pwd
/home/benzhang

```



退出ssh登陆： Ctrl + D

底盘的文件夹中，**nav_ws/ ** 我们代码都在这个空间里



roslaunch gmapping slam_gmapping_puppy.launch 

roslaunch ydlidar lidar.launch





## 分部式运行配置

即roscore所在的master节点在底盘运行，自己编写的功能包在本机运行。需要配置host ip以完成底盘和电脑间的通信。

本机：

<span style='background:aqua'>**~/.bashrc**</span>

![image-20200519113735104](/home/liwenzhi/.config/Typora/typora-user-images/image-20200519113735104.png)

底盘:

benzhang@localhost:~$ sudo vim <span style='background:aqua'>**/etc/hosts**</span>

![image-20200519113930044](/home/liwenzhi/.config/Typora/typora-user-images/image-20200519113930044.png)

ip与用户名之间是Tab





## 启动 minimal 节点



```shell
## --screen参数可以不加, 回显输出到终端
## 该launch文件会启动底盘的 master 节点
benzhang@localhost:~$ roslaunch kobuki_node minimal.launch --screen
```

上面命令在bashrc里重命名为：basestart

![image-20200527165203316](/home/liwenzhi/.config/Typora/typora-user-images/image-20200527165203316.png)

---

**kobuki_node**

kobuki是一个堆，底盘控制的包的集合，开源，基于ROS开发。

---







## 底盘节点Topic

```shell
## 以下为 master 节点的当前所有话题
benzhang@localhost:~$ rostopic list 
/diagnostics
/imu
/joint_states
/keyop/teleop
/left_right
/mobile_base/commands/controller_info
/mobile_base/commands/digital_output
/mobile_base/commands/external_power
/mobile_base/commands/led1
/mobile_base/commands/led2
/mobile_base/commands/motor_power
/mobile_base/commands/motor_pwm
/mobile_base/commands/reset_odometry
/mobile_base/commands/sound
/mobile_base/commands/velocity
/mobile_base/commands/wall_follow
/mobile_base/controller_info
/mobile_base/debug/raw_control_command
/mobile_base/debug/raw_data_command
/mobile_base/debug/raw_data_stream
/mobile_base/events/bumper
/mobile_base/events/button
/mobile_base/events/cliff
/mobile_base/events/digital_input
/mobile_base/events/power_system
/mobile_base/events/robot_state
/mobile_base/events/wheel_drop
/mobile_base/odometry_phy
/mobile_base/sensors/core
/mobile_base/sensors/dock_ir
/mobile_base/sensors/imu_data
/mobile_base/version_info
/mobile_base_nodelet_manager/bond
/odom
/rosout
/rosout_agg
/slip_msg
/tf
benzhang@localhost:~$ 
```



/mobile_base/commands/velocity   速度

### 红外信号

/mobile_base/sensors/dock_ir          底盘读回来的红外信号, 详见2.5节

![image-20200526144923096](/home/liwenzhi/.config/Typora/typora-user-images/image-20200526144923096.png)



### 键盘控制机器

roslaunch kobuki_keyop keyop.launch 

节点启动后可在键盘上利用方向键控制机器前进/旋转，空格停止



可以在自己电脑上ehco机器人方向的消息(只有底盘有速度时才会有回显)

```shell
rostopic echo /mobile_base/commands/velocity
## 此时如果在底盘上启动 kobuki_keyop 包的 keyop.launch, 并用方向键控制机器人, 会有以下输出
## x-前进速度
## y-无意义一直是0 
## z-左右转向角, 右转为负
linear: 
  x: 0.05
  y: 0.0
  z: 0.0
angular: 
  x: 0.0
  y: 0.0
  z: 0.0
---
linear: 
  x: 0.05
  y: 0.0
  z: 0.0
angular: 
  x: 0.0
  y: 0.0
  z: 0.0
---
```

kobuki_keyop节点：

```c
liwenzhi@liwenzhi:~$ rospack find kobuki_keyop 
/opt/ros/kinetic/share/kobuki_keyop
liwenzhi@liwenzhi:~$
```



---

**！！！**

如果在程序中想要持续前进或转向，需要一直向对应话题（/mobile_base/commands/velocity）发布数据。

只对对应话题的消息写一次的话，<span style='color:red'>默认持续 0.6s</span>

键盘控制机器的kobuki_keyop节点，只按一下键盘，这个节点就会一直发数据；但在程序里，想要持续前进或转向的话，**<span style='color:red'> 需要在代码里一直循环写 </span>**。



### 电池

```shell
#电池在core这个Topic下
rostopic echo /mobile_base/sensors/core
```



![image-20200526144521548](/home/liwenzhi/.config/Typora/typora-user-images/image-20200526144521548.png)

正常满电 14.5 - 16.8 (单位V)





## 红外信号

### 信号Topic

```shell
liwenzhi@liwenzhi:~/catkin_ws$ rostopic info /mobile_base/sensors/dock_ir 
Type: kobuki_msgs/DockInfraRed  ## msg Type

Publishers: 
 * /mobile_base_nodelet_manager (http://192.168.1.56:41191/)

Subscribers: None


liwenzhi@liwenzhi:~/catkin_ws$ rosmsg show kobuki_msgs/DockInfraRed
## 无用
uint8 NEAR_LEFT=1
uint8 NEAR_CENTER=2
uint8 NEAR_RIGHT=4
uint8 FAR_LEFT=16
uint8 FAR_CENTER=8
uint8 FAR_RIGHT=32
## /无用
## msg头、序列号和时戳
std_msgs/Header header
  uint32 seq
  time stamp
  string frame_id
## /msg头、序列号和时戳

## data[0] => LEFT
## data[1] => LEFT_CENTER
## data[2] => RIGHT_CENTER
## data[3] => RIGHT
uint8[] data

```

### 调用Topic

Topic		/mobile_base/sensors/dock_ir

msg类	  kobuki_msgs::DockInfraRed

---

![image-20200519120711467](/home/liwenzhi/.config/Typora/typora-user-images/image-20200519120711467.png)

回调函数参数一般写该msg类的指针(ConstPtr)。

```c++
//ros::Subscriber sub = 句柄.subscribe<消息类>("Topic", 消息队列长度, 回调函数);
```



### 底盘红外信号

![image-20200519102632583](/home/liwenzhi/.config/Typora/typora-user-images/image-20200519102632583.png)

机器上每个红外传感器都可以接收桩的4个灯的信号，如果同时收到两个，则叠加(与到一起)。如同时收到 Left(0x1) 和 Left-Center(0x2) 信号，则该传感器实际参数为 0x1 || 0x2 = 0x3。

NEAR信号是指桩的顶部红外发射器，它是个近半圆，覆盖角度广，但距离近，可以作为接近参考。



## 底盘其它信号

---

```shell
rosmsg show kobuki_msgs/SensorState
## 以下信号均为SensorState消息的回显
...
uint16 time_stamp
uint8 bumper
uint8 wheel_drop
uint8 cliff
uint16 left_encoder
uint16 right_encoder
int8 left_pwm
int8 right_pwm
uint8 buttons
uint8 charger
uint8 battery
...
```



\< **Charger**\> 表示充电状态

```c++
kobuki_msgs::SensorStateConstPtr &core
    core->charger
```



![image-20200520153853369](/home/liwenzhi/.config/Typora/typora-user-images/image-20200520153853369.png)

\< **Bumper** \> 表示保险杠碰撞区域

![image-20200520154218078](/home/liwenzhi/.config/Typora/typora-user-images/image-20200520154218078.png)





# 机器人位姿

表示三维坐标系，一般用右手系：

![image-20200522090746511](/home/liwenzhi/.config/Typora/typora-user-images/image-20200522090746511.png)

Version:0.9 StartHTML:0000000105 EndHTML:0000000380 StartFragment:0000000141 EndFragment:0000000344  ![img](https://img-blog.csdnimg.cn/20190827164053851.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E4MTIwNzM0Nzk=,size_16,color_FFFFFF,t_70)



## **右手坐标系定义**

把右手放在原点的位置，使大姆指，食指和中指互成直角，把 **大姆指** 指向 **Z轴** 的正方向，**食指**指向**X轴**的正方向时向时，**中指**所指的方向就是 **Y轴** 的正方向。

通常相对于我们的身体而言

- X -> 朝前
- Y -> 朝左
- Z -> 朝上

## 三维坐标轴旋转定义

对于一个三维空间里面的旋转，可以分解成绕着坐标轴的旋转。旋转的方向使用右手法则定义：

![image-20200522091137499](/home/liwenzhi/.config/Typora/typora-user-images/image-20200522091137499.png)

用右手握住坐标轴，**大拇指** 的方向朝着坐标轴**朝向的正方向**，**四指环绕的方向**定义沿着这个坐标轴**旋转的正方向**

一般来说

- 绕 **<span style='color:red'>Z</span> 轴** 旋转，称之为<span style='color:red'> **航向角**</span>，使用 <span style='color:red'>yaw</span> 表示;
- 绕 **<span style='color:green'>X</span> 轴** 旋转，称之为 <span style='color:green'>**横滚角**</span>，使用 <span style='color:green'>roll</span> 表示;
- 绕 **<span style='color:blue'>Y</span> 轴** 旋转，称之为<span style='color:blue'> **俯仰角**</span>，使用 <span style='color:blue'>pitch</span> 表示;



## ROS常见坐标系

用一个例子说明

初始时刻小车位置如下：

![image-20200522092322487](/home/liwenzhi/.config/Typora/typora-user-images/image-20200522092322487.png)

按照之前坐标轴的定义，将小车的朝向方向作为X轴，正左方标识为Y轴，并将小车所在的位置定义为原点。

在原点插一面小旗子，控制小车行进一段距离：

![image-20200522092416345](/home/liwenzhi/.config/Typora/typora-user-images/image-20200522092416345.png)

这个时候我们能得到三个位置信息：

1. 使用测量工具测量小车相对旗子的位置，在X轴正方向距原点3个单位，在Y轴正方向距原点2个单位
2. 小车安装里程计，记录自己前进3个单位，并向左平移了2个单位
3. 小车使用激光雷达数据与已有地图进行匹配，并结合里程计数据，将数据融合得到小车的位置在X轴正方向3个单位，在Y轴正方向2个单位

在上面的例子中，三个坐标值都相同。但真实情况下，三个坐标值由于测量误差或者其他原因导致坐标值并不相同，然而这三个坐标都用来表示小车中心在空间中的位置，这就引出了不同坐标系



**真实坐标系**

对应第一种位置信息。这是一个理想的坐标系，即我们拥有一种绝对准确的测量方式获得小车相对于地图原点的坐标，但这种坐标系在真实情况下是不存在的。

**里程计坐标系**

对应第二种位置信息。在这个坐标系中得到的测量值通常是基于轮速里程计，视觉里程计或者惯性单元得到的。在这个坐标系中，新坐标值通常是根据前一个时刻坐标得到的，一般使用 <span style='color:red'>**odom**</span> 来表示。

- **优点**: 坐标值是连续的并且以平稳的方式演变，没有离散的跳跃。
- **缺点**: 测量产生的误差会累计。
- **适合**: 短时程相对定位

**地图坐标系**

在这个坐标系中得到坐标值通常是通过传感器的数据重新计算或测量得到的，一般使用 <span style='color:red'>**map**</span> 来表示。

- **优点**: 由于每次得到的坐标点都是重新测量计算得到的，累计误差影响较小
- **缺点**: 坐标数据会有跳变。
- **适合**: 长时程绝对定位





# 尺寸参数



## 桩

宽 15.5



## 底盘 (单位cm)

### 尺寸

直径33

中接收器：3.5，间距1

左右接收器，弦长30

中左接收器，弦长17.5

![image-20200526142503729](/home/liwenzhi/.config/Typora/typora-user-images/image-20200526142503729.png)

---

### 接收器接收角

(测量误差可能较大)

![image-20200529145909613](/home/liwenzhi/.config/Typora/typora-user-images/image-20200529145909613.png)

如图，因为角度相对较小，当前测量条件下的值如上，可能会有误差，仅作参考。

左右接收器在中线两端均有一定角度可以接收信号，中间两个接收器只在外侧有一定接收角。



## 信号区(单位cm/度)

![image-20200529091943256](/home/liwenzhi/.config/Typora/typora-user-images/image-20200529091943256.png)



### Near (0x10)

Near信号360度发射

---

---

【右接收器】

不与其它信号叠加：56

---

【其它】

如与其它信号叠加：65-68

不与其它信号叠加：85    ||    把其它灯糊上，Near信号距离也为85



### Left/Right

单个信号发射角26度

---

---

正对时 \> 300



### Mid

---

单个信号发射角25度。

C_L信号与C_R信号中间有3-5度(单侧)的重叠区域

---

---

【右接收器】

正对时 195

---

【其它】

正对时 165



## 不同位置距离

### 空白区域

Right与Mid信号中间空白区域，机器**圆心**在距离**桩中心点**<span style="color:red"> 210.93 </span>时与两侧信号相切

![image-20200529092931700](/home/liwenzhi/.config/Typora/typora-user-images/image-20200529092931700.png)

因为左右两个接收器相距30，当机器中间接收器连线的垂线通过桩中心点时，如果左右两个接收器刚好处于信号区边界，那么 **桩中心** 到 **左右接收器连线中点** 的距离为 <span style='color:red'>190</span>

![image-20200529093644541](/home/liwenzhi/.config/Typora/typora-user-images/image-20200529093644541.png)

### 右接收器 - R信号边界

【右边界】如下，再左转则右接收器收不到右信号

![image-20200529152006990](/home/liwenzhi/.config/Typora/typora-user-images/image-20200529152006990.png)

【左边界】如下，再右转则右接收器收不到右信号

![image-20200529152317193](/home/liwenzhi/.config/Typora/typora-user-images/image-20200529152317193.png)





# 各种Topic



## 沿墙

【状态】

全局清扫 沿墙开始和结束 topic: /global_clean_task/wall_follow
消息类型:std_msgs::UInt8
字段解析: 1:开始 2：结束

## 全局清扫开始/结束

霍章义 8-7 14:16:13
全局清扫 APP控制【命令】 topic:/mobile_base/commands/app_global_clean消息类型:std_msgs::UInt8
字段解析: 1:开始 2：暂停 3：继续 4：结束

霍章义 8-7 14:17:39
全局清扫完成的反馈【状态】 topic: /wall_area_status
消息类型:std_msgs::UInt8
字段解析: 0:完成



# 问题

## 编译提示脚本应以#!开头

![image-20200818114007490](/home/liwenzhi/.config/Typora/typora-user-images/image-20200818114007490.png)

此为编译时 /devel/lib/docking 下的脚本为空，即编译未产生脚本。(此时cmd编译是不显示错误的)

![image-20200818114613252](/home/liwenzhi/.config/Typora/typora-user-images/image-20200818114613252.png)



解决方案：

1. 删除  **devel/lib**/docking/docking_node
2. 删除  **build/robot_task**/docking/
3. 删除  **devel/lib**/docking/
4. 删除  **devel/lib**/libdocking.so
5. 删除  **devel/share**/docking/
6. 重新编译docking节点

