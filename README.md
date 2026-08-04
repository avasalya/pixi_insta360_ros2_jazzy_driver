# pixi_insta360_ros2_jazzy_driver
Insta360 ROS2 (Jazzy) Driver with Pixi environment


# original sources

Thanks to the authors of original sources.

[insta360_ros_driver](https://github.com/ai4ce/insta360_ros_driver)

[pixi_panda_ros2](https://github.com/lvjonok/pixi_panda_ros2)


# Quick Start

## Prerequisites

Install pixi:
```bash
curl -fsSL https://pixi.sh/install.sh | bash
```

## Workspace Build and Environment Management

This repository provides two ways to build your ROS 2 environment: **Option A (Binary)** via Pixi's Robostack channel, and **Option B (Source Fallback)** if Pixi packages fail to solve or run properly on your local system layout.

### Option A: Install via Pixi Binaries (Recommended)

This tracks system requirements directly in the virtual shell layer without flooding your local `src` folder.

This workspace utilizes `pixi` to isolate dependency variants across environments. 
The driver dependencies (`cv_bridge`, `imu_tools`, `ffmpeg`) are configured natively via Pixi's `robostack-jazzy` channel, removing the need to track or build secondary source submodules.

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/avasalya/pixi_insta360_ros2_jazzy_driver 
cd pixi_insta360_ros2_jazzy_driver

# Setup: install dependencies and build
pixi run -e jazzy360 setup

```

### Build via Pixi Tasks

To build your targeted environment completely, utilize the defined environment flag tasks:

```bash
# Build the Jazzy360 Driver environment 
pixi run -e jazzy360 build

```

### Option B: Install From Source Fallback

If Pixi channels throw package resolution/solving errors on your environment, compile the missing packages inside your local `src/` directory instead.

#### 1. Setup Source Workspace Layout
Ensure your `src` directory structures map these manual repositories exactly:
```bash
cd pixi_insta360_ros2_jazzy_driver

# Create the folder structure if not done
mkdir -p src

cd src

# Clone the missing dependent packages inside src/
git clone -b ros2 https://github.com/ros-perception/vision_opencv.git

git clone -b jazzy https://github.com/CCNYRoboticsLab/imu_tools.git

cd ..
```

#### 2. Re-compile Source Dependencies
Run your isolated `colcon` build script within the target `jazzy360` environment mapping:
```bash
pixi run -e jazzy360 colcon build --symlink-install
```

#### 3. Manual Colcon Selection with Pixi Contexts

If you prefer building a specific workspace package or clearing a particular layout directory, pass your selective `colcon` directives natively through the pixi runner context:

```bash
# Build ONLY the insta360_ros_driver package within the jazzy360 context
pixi run -e jazzy360 colcon build --packages-select insta360_ros_driver --symlink-install

# Clean build a package by forcing CMake reconfiguration
pixi run -e jazzy360 colcon build --packages-select insta360_ros_driver --cmake-clean-first
```

### Running the Driver Nodes

```bash
# Launch decoder node 
pixi run -e jazzy360 ros2 run insta360_ros_driver decoder

# Launch equirectangular node
pixi run -e jazzy360 ros2 run insta360_ros_driver equirectangular_cpp
```



### Running Applications and Interfacing

To launch programs inside the initialized workspace environment context without dropping out of your active shell, run:

```bash
# Example running your driver binary target safely packaged by Pixi
pixi run -e jazzy360 ros2 run insta360_ros_driver decoder
```



## Available Commands

| Command | Description |
|---------|-------------|
| `pixi run -e jazzy360 setup` | Full setup: submodules and build |
| `pixi run -e jazzy360 build` | Build packages with colcon |


## Development

Enter the pixi environment:
```bash
pixi shell -e jazzy360
```