# ROS2智能机器人开发实践

这本书大部分内容都比较基础，而且内容基本上脱胎于ROS2的官网。所以很多基础内容我就不写了。写一些我平时不太常用的功能吧。要不内容实在太多了。



# 1.ROS2核心原理

## 1.1 参数

参数在ros2中类似于配置参数的全局变量。可以通过命令dump出yaml文件或者载入进去。



dump操作

```shellscript
ros2 param dump turtlesim >> param.yaml
```

载入操作

```shellscript
ros2 param load turtlesim  param.yaml
```



## 1.2 DDS

DDS是数据分发服务标准，用于事实系统数据分发/订阅的标准。不论是自动驾驶还是具身智能机器人，都存在大量数据的获取去传输，尤其是视觉数据和点云数据。为了保证系统的实时性响应，ROS2开始引入DDS机制，并且设置统一的中间件接口，去适配不同的DDS服务。不同的DDS性能差距巨大。ROS2可以适配Fastdds，cyclonedds等。



在机器人领域常见的通信模式有四种。

第一种是点对点式，比如TCPIP、REST，Websocket等。这种一般用于节点与节点之间通信。通信前服务要约定好ip地址等信息。如果相关信息更改，就需要更改相关的节点配置信息。

第二种是Broker模式，比如mqtt、kafka。它们统一使用broker来管理连接和服务对接。这样获取服务和提供服务的节点只需要连接broker获取服务即可。但是这样存在性能瓶颈，如果所有服务都高频通过broker通信，就会造成服务堵塞。ROS1的master设计就是这种模式，在ROS2设计中被摒弃。

第三种是广播模式，比如CANbus等，它们是通过广播把所有数据都发布出去，其他节点被迫要去甄别不是自己目标的数据。

第四种是DDS，方式类似于广播，但是加上了domain，这样通信只会在每个独立的域内进行，不同得到域内的节点不用考虑不是自己的数据频繁判断接受浪费系统资源和带宽问题。



引入dds后另外带来的一个改变是qos，可以通过设置qos类配置节点间通信是比如保证数据不丢包还是保证尽快传递可以允许部分丢包等问题。



常用的策略

| DEADLINE    | 节点间每个截止时间内完成一次通信                                                               |
| ----------- | ------------------------------------------------------------------------------ |
| HISTORY     | 设置历史数据的缓存大小                                                                    |
| RELIABILITY | BEST\_EFFORT尽力传输模式，网络情况好时，可以保证数据通畅，但是可能会数据丢失&#xA;RELIABLE可信赖模式，可以在通信中尽量保持数据完整性 |
| DURABILITY  | 针对晚加入节点，保证一定历史数据发送过去                                                           |



qos profile，在/opt/ros/jazzy/include/rmw/rmw/qos\_profiles.h中定义qos各种配置信息

```cpp
// Copyright 2015 Open Source Robotics Foundation, Inc.
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//     http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

#ifndef RMW__QOS_PROFILES_H_
#define RMW__QOS_PROFILES_H_

#ifdef __cplusplus
extern "C"
{
#endif

#include "rmw/types.h"

static const rmw_qos_profile_t rmw_qos_profile_sensor_data =
{
  RMW_QOS_POLICY_HISTORY_KEEP_LAST,
  5,
  RMW_QOS_POLICY_RELIABILITY_BEST_EFFORT,
  RMW_QOS_POLICY_DURABILITY_VOLATILE,
  RMW_QOS_DEADLINE_DEFAULT,
  RMW_QOS_LIFESPAN_DEFAULT,
  RMW_QOS_POLICY_LIVELINESS_SYSTEM_DEFAULT,
  RMW_QOS_LIVELINESS_LEASE_DURATION_DEFAULT,
  false
};

static const rmw_qos_profile_t rmw_qos_profile_parameters =
{
  RMW_QOS_POLICY_HISTORY_KEEP_LAST,
  1000,
  RMW_QOS_POLICY_RELIABILITY_RELIABLE,
  RMW_QOS_POLICY_DURABILITY_VOLATILE,
  RMW_QOS_DEADLINE_DEFAULT,
  RMW_QOS_LIFESPAN_DEFAULT,
  RMW_QOS_POLICY_LIVELINESS_SYSTEM_DEFAULT,
  RMW_QOS_LIVELINESS_LEASE_DURATION_DEFAULT,
  false
};

static const rmw_qos_profile_t rmw_qos_profile_default =
{
  RMW_QOS_POLICY_HISTORY_KEEP_LAST,
  10,
  RMW_QOS_POLICY_RELIABILITY_RELIABLE,
  RMW_QOS_POLICY_DURABILITY_VOLATILE,
  RMW_QOS_DEADLINE_DEFAULT,
  RMW_QOS_LIFESPAN_DEFAULT,
  RMW_QOS_POLICY_LIVELINESS_SYSTEM_DEFAULT,
  RMW_QOS_LIVELINESS_LEASE_DURATION_DEFAULT,
  false
};

static const rmw_qos_profile_t rmw_qos_profile_services_default =
{
  RMW_QOS_POLICY_HISTORY_KEEP_LAST,
  10,
  RMW_QOS_POLICY_RELIABILITY_RELIABLE,
  RMW_QOS_POLICY_DURABILITY_VOLATILE,
  RMW_QOS_DEADLINE_DEFAULT,
  RMW_QOS_LIFESPAN_DEFAULT,
  RMW_QOS_POLICY_LIVELINESS_SYSTEM_DEFAULT,
  RMW_QOS_LIVELINESS_LEASE_DURATION_DEFAULT,
  false
};

static const rmw_qos_profile_t rmw_qos_profile_parameter_events =
{
  RMW_QOS_POLICY_HISTORY_KEEP_LAST,
  1000,
  RMW_QOS_POLICY_RELIABILITY_RELIABLE,
  RMW_QOS_POLICY_DURABILITY_VOLATILE,
  RMW_QOS_DEADLINE_DEFAULT,
  RMW_QOS_LIFESPAN_DEFAULT,
  RMW_QOS_POLICY_LIVELINESS_SYSTEM_DEFAULT,
  RMW_QOS_LIVELINESS_LEASE_DURATION_DEFAULT,
  false
};

static const rmw_qos_profile_t rmw_qos_profile_system_default =
{
  RMW_QOS_POLICY_HISTORY_SYSTEM_DEFAULT,
  RMW_QOS_POLICY_DEPTH_SYSTEM_DEFAULT,
  RMW_QOS_POLICY_RELIABILITY_SYSTEM_DEFAULT,
  RMW_QOS_POLICY_DURABILITY_SYSTEM_DEFAULT,
  RMW_QOS_DEADLINE_DEFAULT,
  RMW_QOS_LIFESPAN_DEFAULT,
  RMW_QOS_POLICY_LIVELINESS_SYSTEM_DEFAULT,
  RMW_QOS_LIVELINESS_LEASE_DURATION_DEFAULT,
  false
};

/// Match majority of endpoints currently available while maintaining the highest level of service
/**
 * Reliability, durability, deadline, liveliness, and liveliness lease duration policies will be
 * chosen at the time of creating a subscription or publisher.
 *
 * The actual QoS policy can be retrieved after the endpoint is created with
 * `rmw_get_subscriptions_info_by_topic` or `rmw_get_publishers_info_by_topic`.
 *
 * The middleware is not expected to update policies after creating a subscription or
 * publisher, even if one or more policies are incompatible with newly discovered endpoints.
 * Therefore, this profile should be used with care since non-deterministic behavior
 * can occur due to races with discovery.
 */
static const rmw_qos_profile_t rmw_qos_profile_best_available =
{
  RMW_QOS_POLICY_HISTORY_KEEP_LAST,
  10,
  RMW_QOS_POLICY_RELIABILITY_BEST_AVAILABLE,
  RMW_QOS_POLICY_DURABILITY_BEST_AVAILABLE,
  RMW_QOS_DEADLINE_BEST_AVAILABLE,
  RMW_QOS_LIFESPAN_DEFAULT,
  RMW_QOS_POLICY_LIVELINESS_BEST_AVAILABLE,
  RMW_QOS_LIVELINESS_LEASE_DURATION_BEST_AVAILABLE,
  false
};

static const rmw_qos_profile_t rmw_qos_profile_unknown =
{
  RMW_QOS_POLICY_HISTORY_UNKNOWN,
  RMW_QOS_POLICY_DEPTH_SYSTEM_DEFAULT,
  RMW_QOS_POLICY_RELIABILITY_UNKNOWN,
  RMW_QOS_POLICY_DURABILITY_UNKNOWN,
  RMW_QOS_DEADLINE_DEFAULT,
  RMW_QOS_LIFESPAN_DEFAULT,
  RMW_QOS_POLICY_LIVELINESS_UNKNOWN,
  RMW_QOS_LIVELINESS_LEASE_DURATION_DEFAULT,
  false
};

typedef enum RMW_PUBLIC_TYPE rmw_qos_compatibility_type_e
{
  /// QoS policies are compatible
  RMW_QOS_COMPATIBILITY_OK = 0,

  /// QoS policies may not be compatible
  RMW_QOS_COMPATIBILITY_WARNING,

  /// QoS policies are not compatible
  RMW_QOS_COMPATIBILITY_ERROR
} rmw_qos_compatibility_type_t;


/// Check if two QoS profiles are compatible.
/**
 * Two QoS profiles are compatible if a publisher and subcription
 * using the QoS policies can communicate with each other.
 *
 * If any of the profile policies has the value "system default" or "unknown", then it may not be
 * possible to determine the compatibilty.
 * In this case, the output parameter `compatibility` is set to `RMW_QOS_COMPATIBILITY_WARNING`
 * and `reason` is populated.
 *
 * If there is a compatibility warning or error, and a buffer is provided for `reason`, then an
 * explanation of all warnings and errors will be populated into the buffer, separated by
 * semi-colons (`;`).
 * Errors will appear before warnings in the string buffer.
 * If the provided buffer is not large enough, this function will still write to the buffer, up to
 * the `reason_size` number of characters.
 * Therefore, it is possible that not all errors and warnings are communicated if the buffer size limit
 * is reached.
 * A buffer size of 2048 should be more than enough to capture all possible errors and warnings.
 *
 * <hr>
 * Attribute          | Adherence
 * ------------------ | -------------
 * Allocates Memory   | No
 * Thread-Safe        | Yes
 * Uses Atomics       | No
 * Lock-Free          | Yes
 *
 * \param[in] publisher_profile: The QoS profile used for a publisher.
 * \param[in] subscription_profile: The QoS profile used for a subscription.
 * \param[out] compatibility: `RMW_QOS_COMPATIBILITY_OK` if the QoS profiles are compatible, or
 *   `RMW_QOS_COMPATIBILITY_WARNING` if the QoS profiles might be compatible, or
 *   `RMW_QOS_COMPATIBILITY_ERROR` if the QoS profiles are not compatible.
 * \param[out] reason: A detailed reason for a QoS incompatibility or potential incompatibility.
 *   Must be pre-allocated by the caller.
 *   This parameter is optional and may be set to `NULL` if the reason information is not
 *   desired.
 * \param[in] reason_size: Size of the string buffer `reason`, if one is provided.
 *   If `reason` is `nullptr`, then this parameter must be zero.
 * \return `RMW_RET_OK` if the check was successful, or
 * \return `RMW_RET_INVALID_ARGUMENT` if `compatibility` is `nullptr`, or
 * \return `RMW_RET_INVALID_ARGUMENT` if `reason` is `NULL` and  `reason_size` is not zero, or
 * \return `RMW_RET_ERROR` if there is an unexpected error.
 */
RMW_PUBLIC
RMW_WARN_UNUSED
rmw_ret_t
rmw_qos_profile_check_compatible(
  const rmw_qos_profile_t publisher_profile,
  const rmw_qos_profile_t subscription_profile,
  rmw_qos_compatibility_type_t * compatibility,
  char * reason,
  size_t reason_size);

#ifdef __cplusplus
}
#endif

#endif  // RMW__QOS_PROFILES_H_

```

有一个坑是说，如果节点间qos设置不匹配是不能正常进行通信的。



DDS带来另外一个特点是分布式通信。可以在同一局域网内跨节点通信。至于不在同一个局域网内，写个网桥，让指定的网桥节点做通信中转节点就行了。



设置环境变量

```shellscript
export ROS_DOMAIN_ID = ID  # ID设置成0-255
```

把不同计算机的ROS2节点连在同一个网络，保证网络正常的情况下，启动节点前配置域id的环境变量就可以进行通信。



另外作为消息中间件，ROS有计划用Zenoh来代替DDS。二者在机器人通信中的优缺点对比如下:



DDS通信依赖udp组播，但是这样的通信方式容易被防火墙、WSL2、docker、vpn、内网等网络设施阻断。那么使用Zenoh可以先通过zenoh-bridge-ros2跟zenoh网络进行通信，而zenoh网络使用tcp/udp/quic等通信协议，真实的通信完全由它来管理，再通过zenoh-bridge-ros2；连接到另外一个ros节点。这样的优点可以跨子网通信，对于wsl2这种虚拟网络也比较优化。



# 2.ROS2常用工具

## 2.1 launch配置

ros2里面一个进程可以运行多个节点，也可以多个进程运行多个节点。每个节点都可以有不同的配置。而一个复杂的机器人软件系统，必然包含大量不同功能的节点、进程、线程、配置等资源。launch就是解决这么多节点启动配置的方案。launch文件本身是用python写的，这块也是python非常擅长的领域，因为它作为胶水语言，核心就是连接不同功能做自动化脚本。



一个demo来同时启动topic的pub和sub

```python
from launch import LaunchDescription
from launch_ros.actions import Node


def generate_launch_description():
    return LaunchDescription(
        [
            Node(package="py_launch_demo", executable="talker"),
            Node(package="py_launch_demo", executable="listener"),
        ]
    )

```

当然因为可以用python写，对于复杂应用，可是写更复杂的逻辑进去。设置参数，启动顺序，各种配置关系等等。



在launch文件夹下写节点启动配置的launch文件后，还需要再setup.py中设置launch文件安装路径

```python
from setuptools import find_packages, setup
import os
from glob import glob

package_name = "py_launch_demo"

setup(
    name=package_name,
    version="0.0.0",
    packages=find_packages(exclude=["test"]),
    data_files=[
        ("share/ament_index/resource_index/packages", ["resource/" + package_name]),
        ("share/" + package_name, ["package.xml"]),
        (
            os.path.join("share", package_name, "launch"),
            glob(os.path.join("launch", "launch.py")),
        ),
    ],
    install_requires=["setuptools"],
    zip_safe=True,
    maintainer="gaoming",
    maintainer_email="2427373908@qq.com",
    description="TODO: Package description",
    license="TODO: License declaration",
    extras_require={
        "test": [
            "pytest",
        ],
    },
    entry_points={
        "console_scripts": [
            "talker = py_launch_demo.publisher_member_function:main",
            "listener = py_launch_demo.subscriber_member_function:main",
        ],
    },
)

```

启动的时候直接使用ros2的launch命令

```shellscript
 ros2 launch py_launch_demo launch.py
```



除此之外还可以对引用节点的消息进行remap或者添加命名空间。这个设计的初衷是因为节点多了，可能消息有重名，冲突之类的。但是又不能频繁更改代码。，可以通过更改launch文件来把消息remap进而不更改节点代码运行整个复杂机器人系统。而命名空间是防止命名冲突，防止导致消息错配的一种方式。



相关代码地址:

[\[module\]: add py launch module · mingtiancai/learn\_ros2\_all\_you\_need@d5f8194](https://github.com/mingtiancai/learn_ros2_all_you_need/commit/d5f81940d36095549363c3b9c5c13195945690a5#diff-cb07047ecbc6637416be0279eb48cb5eb8f01a60cb3aa9456b14eb855007a7e4)



## 2.2 tf坐标管理

机器人坐标系管理系统。在机器人中如果是关节，一般是六轴机械臂或者七轴机械臂。如果是灵巧手，一般有几个到几十个主动关节。如果是移动机器人，也存在视觉、本体等多个坐标系的变换。所以就需要一种管理坐标系转换的工具。

像机械臂就有基坐标系，世界坐标系，工具坐标系，目标坐标系等。移动机器人有位于自己中心得到基坐标系，雷达的雷达坐标系，里程计的里程计坐标系。地图的地图坐标系。坐标系的变换，因为普遍意义都是三维空间，就是旋转矩阵四元数那些东西。学过机器人或者图形学物理学的都知道，Eigen里面也有现成的接口吗，这里不再赘述。



坐标转换本身存在静态坐标和动态坐标。静态的意思是位姿相对不变。比如安装在移动机器人上的激光雷达。动态坐标就是会随着机器人运动的变化。比如移动机器人的世界坐标。



原书里面的代码存在点问题，我自己改了一下,python实现代码地址



[https://github.com/mingtiancai/learn\_ros2\_all\_you\_need/tree/main/src/py\_tf2\_static\_demo](file:///workspace/AHQ8KS56BzCKekLqcfm9D/RQQKgPvijm)

需要额外安装一个库transforms3d



cpp实现代码地址

[https://github.com/mingtiancai/learn\_ros2\_all\_you\_need/tree/main/src/cpp\_tf2\_static\_demo](file:///workspace/AHQ8KS56BzCKekLqcfm9D/V1kZxwQbYL)

以cpp为例，如果启动节点

```shellscript
ros2 run cpp_tf2_static_demo cpp_tf2_static_demo
```

因为发送坐标是在构造函数中发了一次，后面节点自旋，但是tf2会锁存这次数据，当有新的订阅者要获取这个数据，只要这个节点在，其他订阅节点就可以获取这个坐标广播信息。

比如在另外一个终端使用tf2工具

```shellscript
 ros2 run tf2_tools view_frames
```

就可以看到不同坐标系之间的转换关系，会存成一个pdf文件，里面是图片。



动态坐标广播是对于一些运动机器人，她们坐标随着运动变化，这时候机器人发送位姿的坐标topic，写一个节点订阅，并且用tf2的动态广播播放这个坐标。当然直接绕过tf2获取坐标信息自己解析也可以。只是这样就不能使用tf2自带的一些工具了。差别仅此而已。



tf2也有监听坐标信息的函数接口。



## 2.3 Gazebo仿真

Gazebo是ros中的仿真器。在ubuntu24 ros2 jazzy下安装

```shellscript
sudo apt-get update
sudo apt-get install curl lsb-release gnupg

sudo curl https://packages.osrfoundation.org/gazebo.gpg --output /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] https://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] https://packages.osrfoundation.org/gazebo/ubuntu-prerelease $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/gazebo-prerelease.list > /dev/null
sudo apt-get update
sudo apt-get install gz-jetty
```



启动方式

```shellscript
gz sim
```



仿真器的意义是在软件算法进行真机部署前，先跑一下看看效果，别出现效果非常差劲甚至错误，等到真机上才发现。因为机器人也好，自动驾驶，无人机也罢，真机往往都是嵌入式的，调试成本比较大。所以能通过仿真拦截的问题就不要拖到真机。另外现在机器人非常的智能水平依赖AI，而目前的AI都是基于学习算法的。因为在AI模型部署前，通过大量仿真对模型进行评测非常重要。



## 2.4 RViz可视化

RViz当前的主流版本是RViz2，是用qt写的可视化3d模型，传感器数据等各种信息的GUI软件。这个主要用于调试。让节点按照ROS中对应消息发布数据，就可以在上面接入数据显示。



启动命令

```shellscript
ros2 run rviz2 rviz2
```



## 2.5 rosbag

记录机器人或者传感器的数据状态信息，通过对通信数据尽心记录落盘成文件，也可以进行播放。



## 2.6 rqt模块可视化

rqt也是一个模块化可视化工具，是基于qt开发的。当然它可以去看各种消息的更新情况。

启动命令

```shellscript
rqt
```

可以看日志显示信息啥的。



# 3.其他

后面几章内容其实大部分要么是电机控制，我平时没过过，要么是上下位机通信、计算机视觉、SLAM建图之类的我都很熟悉而且在工作中做出过在业内有点影响的产品。所以我不会很细致的写。都归类到这里了。



## 3.1 控制

常见的机器人主要是通过电机来控制的。这里面机器人上的和自动驾驶或者无人机都非常类似。基本分为两类控制器。一类是域控制器，这类开发平台比如RDK，NVIDIA ORIN，树莓派等。其实就类似一个小电脑了。我就把我的orin开发板当小电脑用，在上面实验各种软件功能。尤其英伟达的开发板带cuda，可以进行AI计算。RDK是国内地平线公司推出的开发板，也有可以进行AI计算的SDK可以调用。一般在域控上跑SLAM，视觉算法，VLA等复杂需要大算力的应用程序。当然这种域控的系统也可以刷成RTOS或者打RTOS补丁。另外一种算力比较小，用单片机或者微控制器，叫做运动控制器。主要功能就是控制电路信号来控制电机的运行。比如STM32或者各种德州仪器的微控制器。运动控制器是纯嵌入式的活。域控主要也是嵌入式，不过可以跑linux。像我之前做的机器人，就是ubuntu+rt补丁，cpu是ARM的芯片。这也是最常规的。



上一章所讲的RViz或者一些上位机软件是要显示传感器信息或者机器人3d的渲染，这种放在一个单独工作的电脑上，跟域控制器通过网络连接，可以使用ros2的消息，也可以自己定义或者使用公开的通信协议进行通信。域控和MCU之前，我在自动驾驶上是用CAN通信，这个不绝对，我以前在其他机器人上也使用工业以太网等。



电机的核心功能就是转动。通过脉冲宽度调制，可以设置方波的占用比，进而控制电机转速。获取电机转速的方式常见有光电码盘式编码器或者霍尔式编码器，基本原理都是通过高速的采样，获得电机转动的物理信息，来推算出电机转速。相当于编程里面Set和Get方法。只有具备这个条件才能在更上层去实现控制算法。



控制算法肯定要闭环，常规的就PID，当然一些现代的控制方法比如LQR,MPC等也有。PID算法的实现在代码层面其实比较简单。当然实际应用是另外一码事，需要针对具体场景调参。



正逆学解在关节里面比较常见了。直到最终的目标，解算各个控制电机要运动的角度叫逆运动学。知道各个电机的关节角度解算目标的位姿是正运动学。常规的机械臂的正逆运动学很完善了。现在比较火的灵巧手因为其硬件设计远比机械臂复杂，正逆运动学求解也要复杂很多。



这本书里面在域控和运控之间通信使用的是串口，但是实际上串口通信RS485的通信效率比较低，在实时性上不如can和以太网。他这里自定义的串口协议就不介绍了。



## 3.2 感知驱动节点

机器人想获取外部的环境信息就需要用到感知传感器，比如相机，激光雷达，IMU等。ros2中有专门的usb相机驱动节点

```shellscript
sudo apt install ros-jazzy-usb-cam
```



| 原始格式图像消息 | sensor\_msgs/msg/Image       |
| -------- | ---------------------------- |
| 压缩图像消息   | sensor\_msgs/CompressedImage |
| 点云消息     | sensor\_msgs/msg/PointCloud2 |
| 雷达消息     | sensor\_msgs/msg/LaserScan   |
| IMU      | sensor\_msgs/msg/Imu         |



机器人视觉或者说机器视觉、计算机视觉，我一直都在这个行业里做项目落地，已经非常熟悉了，所以这部分不写了。



## 3.3 AI推理部署

书里面说的部署工具主要是TensorRT，当然我以前实验室师兄作为技术负责人推出的PPL也算国产之光。书里面说的AI模型只要还是经典的Ai机器视觉模型。大模型出来之后，模型部署工具也有了更新，有专门针对大模型推理部署的工具比如tensorRT-llm等。而且大模型尤其VLA对于机器人当前影响很大，所以这块书里没怎么写哈哈。



第八章讲的是SLAM，这个我以前做过激光slam。所以这块不写了。



## 3.4 自主导航

在导航这块书里讲了一个是导航工具Nav2以及行为树。这个行为树在机器人里面应用十分广泛，主要用来调度不同算法。通过设置好的xml文件。Nav2主要用于移动机器人导航和仿真。细节就不说了，不是做机器人导航必须得软件。









