# Allegro Hand V6 F — ROS 2 Jazzy

ROS 2 software for the **Allegro Hand V6 F (Force model)**, a five-finger robotic hand with 20 actuated joints. This repository includes the hardware driver, robot models, controller bringup, and UI applications in one workspace.

**Version: 1.0.0** · **Ubuntu 24.04 LTS / ROS 2 Jazzy**

Release versions use semantic versioning and Git tags such as `v1.0.0`. See [CHANGELOG.md](CHANGELOG.md) for release history.

## F and V Models

The F and V models share the same control software. Select the model that matches your physical hand when launching bringup:

| Model | Bringup argument | Vision sensor mapping |
|---|---|---|
| **V6 F — Force** (this repository) | `model:=v6_f` | **Not required.** Proceed directly to bringup and the UI. |
| V6 V — Vision | `model:=v6_v` | Follow the vision sensor guide when using the finger cameras. |

All examples below use the **F model, right hand**. For a left hand, use `hand:=left` in both bringup and the dashboard. The shared UI documentation also covers V-model cameras; those capture and mapping steps do not apply to the F model.

## Documentation

| Guide | Contents |
|---|---|
| [Bringup and hardware setup](src/allegro_hand_v6/allegro_hand_v6_bringup/README.md) | Ethernet setup, mock and physical operation, launch arguments, and controller checks. |
| [UI usage](src/allegro_hand_v6_ui/README.md) | Dashboard, pose control GUI, browser monitor, and their ROS interfaces. |
| [Dashboard manual](src/allegro_hand_v6_ui/allegro_hand_v6_dashboard/README.md) | Visualization, pressure display, layouts, and dashboard settings. |
| [CSV recording guide (Korean)](src/allegro_hand_v6/allegro_hand_v6_bringup/CSV_LOGGING.md) | Recording options, data columns, and rate checks. |
| [Validation record](VALIDATION.md) | Recorded build and mock checks, source refreshes, and verification scope. |

## Installation

Install ROS 2 Jazzy Desktop on Ubuntu 24.04 before continuing. The base packages and UI are included here; **build them together in this workspace**. The UI README also describes a separate UI distribution, whose overlay installation steps are unnecessary for this combined repository.

### 1. Clone the Repository

```bash
git clone https://github.com/Wonikrobotics-git/allegro_hand_ros2_v6_f.git \
  ~/allegro_hand_ros2_v6_f
cd ~/allegro_hand_ros2_v6_f
```

### 2. Install Dependencies

In a new terminal:

```bash
source /opt/ros/jazzy/setup.bash
sudo apt update
sudo apt install -y build-essential python3-colcon-common-extensions python3-rosdep

# Initialize rosdep once per PC, if it has not already been initialized.
if [ ! -f /etc/ros/rosdep/sources.list.d/20-default.list ]; then
  sudo rosdep init
fi
rosdep update --rosdistro jazzy

cd ~/allegro_hand_ros2_v6_f
rosdep install --from-paths src --ignore-src --rosdistro jazzy -r -y
```

### 3. Build

```bash
cd ~/allegro_hand_ros2_v6_f
colcon build --symlink-install --executor sequential --cmake-args -DBUILD_TESTING=OFF
source install/setup.bash
```

Use Jazzy environments throughout. Load the following in **every terminal** used to run the hand or UI:

```bash
source /opt/ros/jazzy/setup.bash
source ~/allegro_hand_ros2_v6_f/install/setup.bash
```

## Quick Start Without Hardware

Start mock bringup in terminal 1:

```bash
ros2 launch allegro_hand_v6_bringup allegro_hand.launch.py \
  model:=v6_f hand:=right \
  ros2_control_hardware_type:=mock_components gui:=false
```

Open the dashboard in terminal 2:

```bash
ros2 launch allegro_hand_v6_dashboard dashboard.launch.py hand:=right
```

The dashboard displays the robot model and joint feedback from bringup. Its `hand` argument selects the 2D hand layout; the 3D model comes from bringup. Physical pressure readings require a connected hand. **No vision sensor mapping is needed for the F model.**

## Run with a Physical Hand

Follow the [hardware setup guide](src/allegro_hand_v6/allegro_hand_v6_bringup/README.md#hardware-setup) to connect power and Ethernet and configure the PC's network address. Stop mock bringup before starting the hardware driver.

In terminal 1, use the actual IP address and port of your hand. The following command activates the physical hardware interface:

```bash
ros2 launch allegro_hand_v6_bringup allegro_hand.launch.py \
  model:=v6_f hand:=right \
  ros2_control_hardware_type:=hardware \
  io_interface_descriptor:=modbus_tcp:192.168.1.100:502 \
  gui:=false
```

In terminal 2, launch the dashboard as shown above. Sending poses from the pose GUI commands real joints.

For pose saving, sequences, and browser plots, see [UI usage](src/allegro_hand_v6_ui/README.md#optional-pose-control-gui). Use the same hand setting and `ROS_DOMAIN_ID` across applications.

## Optional CSV Recording

Add the following arguments to the physical bringup command:

```text
csv_logging:=true csv_directory:=./data_logs
```

Recording is disabled by default. CSV files contain timestamps, status counters, 20 joint positions, 20 measured currents, and 18 pressure values. The configured control rate is **500 Hz**; actual acquisition rate depends on the device, transport, and PC. See the [CSV recording guide](src/allegro_hand_v6/allegro_hand_v6_bringup/CSV_LOGGING.md) for details.

## Repository Layout

```text
src/
├── allegro_hand_common/   # Shared messages, I/O, utilities, and controllers
├── allegro_hand_v6/       # Bringup, robot descriptions, and hardware driver
└── allegro_hand_v6_ui/    # Dashboard, pose GUI, browser monitor, and vision tools
```

The vision tools are retained as part of the shared software suite. F-model operation does not require camera discovery or finger mapping.
