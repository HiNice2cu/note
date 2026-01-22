# ROS 基础流程

# 一、学习前准备

1. **安装系统**：Ubuntu（可虚拟机安装）

2. **安装 ROS**：按教程安装对应版本

3. **安装 VSCode**：并安装 ROS 和 C++ 插件，否则无法补全代码

# 二、ROS 开发流程（八步）

## 第一步：创建工作空间

- 创建根文件夹（如 `my_ws`）

- 在其下创建 `src` 文件夹（存放代码）

## 第二步：创建功能包

- 进入 `src` 目录

- 执行命令：

```bash

catkin_create_pkg 包名 roscpp
```

（依赖选 `roscpp`，其他后期再加）

## 第三步：编写简单代码

- 在功能包的 `src` 文件夹下创建 `.cpp` 文件

- 固定开头：

```cpp

#include <ros/ros.h>
int main(int argc, char **argv) {
    ros::init(argc, argv, "节点名");
    ros::NodeHandle nh;
    ROS_INFO("Hello, ROS!");
    return 0;
}
```

## 第四步：设置编译规则

- 打开功能包中的 `CMakeLists.txt`

- 添加：

```cmake

add_executable(可执行文件名 src/你的程序.cpp)
target_link_libraries(可执行文件名 ${catkin_LIBRARIES})
```

## 第五步：编译

- 回到工作空间根目录

- 执行：

```bash

catkin_make
```

生成的可执行文件在`devel` 文件夹中

## 第六步：生效环境变量

临时生效（当前终端）：

```bash

source devel/setup.bash
```

## 第七步：运行节点

1. 启动 ROS 核心（任意终端）：

```bash

roscore
```

1. 运行节点：

```bash

rosrun 包名 可执行文件名
```

## 第八步：循环与话题通信

### 循环结构（固定写法）：

```cpp

ros::Rate loop_rate(10); // 10Hz
while (ros::ok()) {
    // 你的代码
    loop_rate.sleep();
}
```

### 发布者（说话的人）：

```cpp

ros::Publisher pub = nh.advertise<消息类型>("话题名", 10);
// 发布消息：
pub.publish(msg);
```

### 订阅者（听话的人）：

```cpp

void callback(const 消息类型::ConstPtr& msg) {
    // 处理消息
}
ros::Subscriber sub = nh.subscribe("话题名", 10, callback);
// 在循环后加：
ros::spinOnce();  // 或 ros::spin();
```

# 三、常用命令

- `rostopic list`：查看所有话题

- `rostopic echo /话题名`：查看话题消息

- `rosnode list`：查看运行中的节点

- `rosnode info /节点名`：查看节点详情

# 四、注意

- 新开终端要重新 `source devel/setup.bash`

- 先启动 `roscore` 再运行节点

- C++ 需要编译，Python 不需要

- 功能包 = 代码模块，放在 `src` 下