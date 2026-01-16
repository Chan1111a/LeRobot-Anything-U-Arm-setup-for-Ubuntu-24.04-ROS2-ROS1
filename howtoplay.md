# 🔧 System Setup

## OS / ROS Matrix

- **Ubuntu 24.04 + ROS 2 Jazzy (host)**: use this for new installs. The teleop nodes in this repo are still ROS1; run them in a Noetic container or port them to ROS2.
- **Ubuntu 20.04 + ROS1 Noetic (legacy)**: original native environment. Kept below for reference.

---

## Ubuntu 24.04 + ROS 2 Jazzy + Conda

1. **Install ROS 2 Jazzy (system)**

   ```sh
   sudo apt update
   sudo apt install -y software-properties-common curl
   sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
     -o /usr/share/keyrings/ros-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
   sudo apt update
   sudo apt install -y ros-jazzy-desktop ros-dev-tools python3-colcon-common-extensions
   ```

2. **Create and activate a Conda env**

   ```sh
   # install mamba/conda first if you don't have it
   mamba create -n uarm-ros2 python=3.12 pip
   mamba activate uarm-ros2
   ```

3. **Install Python dependencies (non-ROS)**

   - `cv-bridge` 在 PyPI 无 Jazzy 版本，需过滤；`pyrealsense2` 旧版本不支持 Py3.12，换用 2.56.5.9235（或无相机可跳过）。
   ```sh
   # 生成适配 24.04/py3.12 的依赖清单
   grep -v '^cv-bridge' requirements.txt \
     | sed 's/^pyrealsense2==2.55.1.6486/pyrealsense2==2.56.5.9235/' \
     | sed 's/^scipy==1.10.1/scipy==1.11.4/' \
     > /tmp/requirements-nocv.txt
   pip install -r /tmp/requirements-nocv.txt

   # 如需在宿主 ROS2 环境使用 cv_bridge（可选）
   sudo apt install -y ros-jazzy-cv-bridge
   ```

4. **Source ROS 2 inside the env when working**

   ```sh
   source /opt/ros/jazzy/setup.bash
   ```

5. **Run the ROS1 teleop stack from a Noetic container (required until ROS2 nodes exist)**

   The codebase uses ROS1 nodes. On Ubuntu 24.04, launch them in a Noetic container with host networking and device access, then follow the ROS1 steps below inside the container:

   ```sh
   docker run --rm -it --net host --privileged --name uarm-noetic \
     -v /dev:/dev \
     -v $(pwd):/work \
     osrf/ros:noetic-desktop-full \
     bash -lc "cd /work && pip install -r overall_requirements.txt && catkin_make && bash"
   ```

   Inside the container:
   ```sh
   source devel/setup.bash
   roscore
   # open another shell in the same container (docker exec -it uarm-noetic bash)
   # and run the ROS1 teleop commands listed below
   ```

   > If you need ROS2 consumers on the host, bridge topics from the container with `ros1_bridge` (build it inside the container with both workspaces sourced).

---

## Legacy ROS1 (Ubuntu 20.04 / Noetic)

1. **Install Python Dependencies**

   ```sh
   # install both ROS1 and simulation requirements
   pip install -r overall_requirements.txt
   ```
   
   If your system doesn't support ROS1, you can install the dependencies without ROS1 with the following command which supports simulation teleoperation and check [this note](https://github.com/MINT-SJTU/Lerobot-Anything-U-arm/blob/main/src/simulation/README.md).
   ```sh
   pip install -r requirements.txt
   ```

2. **Build Catkin Workspace**

   ```sh
   catkin_make
   source devel/setup.bash
   ```

3. **Verify Installation**

   ```sh
   # Test if ROS can find the package
   rospack find uarm
   ```

---

# 🤖 Plug-and-Play with Real Robot with ROS1
> Zhonglin servo version
## 1. Start ROS Core

Open a terminal and run:

```sh
roscore
```

## 2. Verify Teleop Arm Output

In a new terminal, check servo readings:

```sh
rosrun uarm servo_zero.py
```

This will display real-time angles from all servos. You should check whether `SERIAL_PORT` is available on your device and modify the variable if necessary. 

## 3. Publish Teleop Data

Still in the second terminal, start the teleop publisher:

```sh
rosrun uarm servo_reader.py
```

Your teleop arm now publishes to the `/servo_angles` topic.

## 4. Control the Follower Arm

Choose your robot and run the corresponding script:

- **For Dobot CR5:**
  ```sh
  rosrun uarm scripts/Follower_Arm/Dobot/servo2Dobot.py
  ```

- **For xArm:**
  ```sh
  rosrun uarm scripts/Follower_Arm/xarm/servo2xarm.py
  ```

---

> Feetech servo version (Global Version)
## 1. Start ROS Core

Open a terminal and run:

```sh
roscore
```

## 2. Verify Teleop Arm Output

In a new terminal, check servo readings:

```sh
rosrun uarm feetech_servo_zero.py
```

This will display real-time angles from all servos. You should check whether `SERIAL_PORT` is available on your device and modify the variable if necessary. You may find all servos' angles are `2047` since servo's position is set as `2047` (0~4095 for $360^\circ$) when this code starts running.

## 3. Publish Teleop Data

Still in the second terminal, start the teleop publisher:

```sh
rosrun uarm feetech_servo_reader.py
```

Servo's position is set again as `2047` when this code starts running . **Please return UARM to initial position before starting this script.** Your teleop arm now publishes to the `/servo_angles` topic.

## 4. Control the Follower Arm

Choose your robot and run the corresponding script:

- **For Dobot CR5:**
  ```sh
  rosrun uarm scripts/Follower_Arm/Dobot/servo2Dobot.py
  ```

- **For xArm:**
  ```sh
  rosrun uarm scripts/Follower_Arm/xarm/servo2xarm.py
  ```
---

# 🖥️ Try It Out in Simulation

If you do not have robot hardware, you can try teleoperation in simulation.  
See detailed guidance [here](https://github.com/MINT-SJTU/Lerobot-Anything-U-arm/blob/feat/simulation/src/simulation/README.md).
