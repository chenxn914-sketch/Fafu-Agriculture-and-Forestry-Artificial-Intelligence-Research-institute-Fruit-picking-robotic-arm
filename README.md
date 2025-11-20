# Fafu-Agriculture-and-Forestry-Artificial-Intelligence-Research-institute-Fruit-picking-robotic-arm
基于Ubuntu22.04 ros2 humble版本的水果采摘机械臂，六关节自由度，通过moveit进行路径规划和矩阵变换得到各关节变化的角度信息。

## 使用说明

 

本说明介绍如何使用本项目中的启动机械臂demo，开启机械臂驱动、爪子驱动，开启摄像头识别橘子

本机械臂的手臂arm规划组的驱动与夹爪的驱动是分开启动的

驱动板上1-7号串口是arm规划组，9、10号串口是单独夹爪旋转+抓取驱动

 

### 1. 构建与环境设置

首次或更新源码后进行编译，并在当前终端加载环境：

 

```bash

cd /home/jetson/robot_lvbu_arm_NO2

colcon build --symlink-install

source /home/jetson/robot_lvbu_arm_NO2/install/setup.bash

```


### 2. 开启并初始化机器人状态

1) 启动机械臂rviz的demo（根据设备环境脚本）：

 

```bash

source /home/jetson/robot_lvbu_arm_NO2/install/setup.bash

ros2 launch alica_moveit_controller_newest demo.launch.py

```
运行效果如下
---
<img width="815" height="467" alt="1763624359318" src="https://github.com/user-attachments/assets/21e5695b-74c3-459e-89fd-f3d90407fdd6" />
<img width="819" height="611" alt="image" src="https://github.com/user-attachments/assets/a4b4490d-8c69-4090-a951-8a92f26d5b8e" />

 
2) 再打开机械臂arm规划组的驱动程序：(这是自己写的驱动程序)
 
```bash

ros2 launch serial_arm_driver_cpp serial_arm_driver_launch.py

```


3) 打开三块驱动板上的电源

此时机械臂会直接达到rviz上的初始姿态(如果此步骤先于前1、2两步，会先达到舵机初始姿态，再打开demo和机械臂驱动程序才会到达rviz上机械臂的姿态)
---

 
### 3. 启动摄像头开启yolo识别橘子坐标(以下二选一)(使用的摄像头设备是英特尔的D435)
 
1）(不是脑电远程)

```bash

ros2 launch  orange_detect_python orange_detect_launch.py

```
<img width="816" height="459" alt="image" src="https://github.com/user-attachments/assets/efc52d61-1c03-47c8-ba9a-b89aab71f829" />


2）(脑电远程模式)

```bash

ros2 run  orange_detector_pkg detector_node

```


### 4. 开启旋转爪子和抓取驱动程序

```bash

ros2 run lvbu_moveit_controller target_pose_listener

```


### 5. 启动机械臂规划程序

 

```bash

ros2 launch lvbu_moveit_controller alica_moveit_control_improved.launch.py

```

alica_moveit_control_improved.launch.py此程序是机械臂的核心程序，包括了其订阅摄像头发布的橘子坐标，包含机械臂规划算法和抓取的一些控制逻辑，在此launch文件中可以对传过来坐标值xyz进行修改，对其default_value值进行修改，如何就可以目标值进行一个修正处理

 

    x_offset_arg = DeclareLaunchArgument(

        'x_offset',

        default_value='0.0',

        description='X轴位置偏移量 (米)'

    )

    

    y_offset_arg = DeclareLaunchArgument(

        'y_offset',

        default_value='0.0',

        description='Y轴位置偏移量 (米)'

    )

    

    z_offset_arg = DeclareLaunchArgument(

        'z_offset',

        default_value='0.0',

        description='Z轴位置偏移量 (米)'

    )

 

使用注意说明:

1、如果要进行爪橘子测试先启动以上的前三点的步骤，确认抓取的橘子后，在启动第四、第五点

2、在进行测试时，要时刻注意第三关节的舵机温度，不要一次性测试超过半小时，以免出现过热情况。

3、如果以上所有程序启动，并没有发现机械臂开始运动，在杜邦线没断的情况下，查看这个程序alica_moveit_control_improved.launch.py的终端，可能是规划失败。
