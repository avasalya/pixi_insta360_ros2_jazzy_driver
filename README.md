# pixi_insta360_ros2_jazzy_driver
Insta360 ROS2 (Jazzy) Driver with Pixi environment

# original sources

Thanks to the authors of original sources.

[insta360_ros_driver](https://github.com/ai4ce/insta360_ros_driver)

[pixi_panda_ros2](https://github.com/lvjonok/pixi_panda_ros2)


# Quick Start

## Installation and Workspace Setup

This project uses `pixi` alongside `robostack-jazzy` binaries to handle systemic dependencies natively. 

The **[insta360_ros_driver (forked)](https://github.com/avasalya/insta360_ros_driver/tree/ros2)** package is tracked directly inside this repository as an active development Git submodule. Dependencies required by the driver (such as `cv_bridge`, `imu_tools`, and `ffmpeg`) are configured natively via Pixi's package resolution layers—**there is no need to manually clone or compile any secondary source packages inside your `src/` workspace directory.** (unless you need to add more features)

## Workspace Build and Environment Management

This repository provides two ways to build your ROS 2 environment: **Option A (Binary)** via Pixi's Robostack channel, and **Option B (Source Fallback)** if Pixi packages fail to solve or run properly on your local system layout.

---

## Mandatory Hardware Setup: Install Proprietary Insta360 SDK
Because the official Insta360 Camera SDK contains proprietary binaries, it cannot be packaged or downloaded automatically via Pixi or Git. You must request the official Linux SDK directly from [Insta360 SDK](https://www.insta360.com/developer/home). 

*Note: Ensure you are using the latest SDK build (released after April 23, 2025).*

Once acquired, manually copy the proprietary headers and library objects into your local submodule folder structures exactly as specified below **before running any build tasks**:

1. **Header Files:** Place the `camera` and `stream` tracking header files straight into the submodule's include directory:
   ```text
   src/insta360_ros_driver/include/
   ```
2. **Binary Libraries:** Place the pre-compiled `libCameraSDK.so` library binary straight into the submodule's lib directory:
   ```text
   src/insta360_ros_driver/lib/
   ```

---

### Option A: Install via Pixi Binaries (Recommended)

### Prerequisites

Install pixi:
```bash
curl -fsSL https://pixi.sh/install.sh | bash
```

This tracks system requirements directly in the virtual shell layer without flooding your local `src` folder.

This workspace utilizes `pixi` to isolate dependency variants across environments. 
The driver dependencies (`cv_bridge`, `imu_tools`, `ffmpeg`) are configured natively via Pixi's `robostack-jazzy` channel, removing the need to track or build secondary source submodules.

*Note: If you already cloned the repository without the `--recurse-submodules` flag, initialize the driver source folder manually before running the build step:*
```bash
git submodule update --init --recursive
```

otherwise clone it with submodules:

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/avasalya/pixi_insta360_ros2_jazzy_driver 
cd pixi_insta360_ros2_jazzy_driver

# Setup: install dependencies and build
pixi run -e jazzy360 setup
```

### Build via Pixi Tasks (optional)

To build your targeted environment completely, utilize the defined environment flag tasks:

```bash
# Build the Jazzy360 Driver environment 
pixi run -e jazzy360 build
```

---

### Option B: Install From Source Fallback

If Pixi channels throw package resolution/solving errors on your environment, compile the missing packages inside your local `src/` directory instead.

#### 1. Setup Source Workspace Layout
Ensure your `src` directory structures map these manual repositories exactly:
```bash
cd pixi_insta360_ros2_jazzy_driver/src


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

If you are iteratively testing code modifications within the `insta360_ros_driver` package and want to rebuild or pass custom compiler commands, execute them directly through the environment task runner context:

```bash
# Build ONLY the insta360_ros_driver package within the jazzy360 context
pixi run -e jazzy360 colcon build --packages-select insta360_ros_driver --symlink-install

# Force Clean build a package by forcing CMake reconfiguration
pixi run -e jazzy360 colcon build --packages-select insta360_ros_driver --cmake-clean-first
```

---

# Running the INSTA360 ROS2 Nodes

For detailed camera configuration, hardware permissions, and device rules mapping (such as setting up `/dev/insta` udev triggers), follow instructions @ [Setup Insta360 Camera](https://github.com/avasalya/insta360_ros_driver/tree/ros2#setup-insta360-camera)



## Available Commands

| Command | Description |
|---------|-------------|
| `pixi run -e jazzy360 setup` | Full setup: submodules and build |
| `pixi run -e jazzy360 build` | Build packages with colcon |

```bash
# Launch decoder node 
pixi run -e jazzy360 ros2 run insta360_ros_driver decoder

# Launch equirectangular node
pixi run -e jazzy360 ros2 run insta360_ros_driver equirectangular_cpp
```


# Pixi Env Development

Enter the pixi environment:
```bash
pixi shell -e jazzy360
```
or, add this bash shorcut to your ~/.bashrc

```bash
pxenv() {
    local env_name="${1:-jazzy360}"
    local manifest="$HOME/pixi_insta360_ros2_jazzy_driver/pixi.toml"

    if [ ! -f "$manifest" ]; then
        echo "Error: Manifest file not found at $manifest"
        return 1
    fi

    # Environment variables current directory context me load karega
    eval "$(pixi shell-hook --manifest-path "$manifest" -e "$env_name")"
}
```