# 🖐️ Allegro Hand V6 F

[![ROS 2: Jazzy](https://img.shields.io/badge/ROS_2-Jazzy-22314E)](https://docs.ros.org/en/jazzy/)
[![Ubuntu: 24.04](https://img.shields.io/badge/Ubuntu-24.04-E95420)](#installation)
[![Version: 1.0.0](https://img.shields.io/badge/Version-1.0.0-2563EB)](VERSION)
[![License: MIT](https://img.shields.io/badge/License-MIT-22C55E)](LICENSE)

**Official ROS 2 software for the Allegro Hand V6 F.**

Control a five-finger hand with **20 actuated joints**, visualize live feedback, and work with desktop pose tools and CSV recording. The hardware driver, robot models, bringup, and UI applications are included in one workspace for **Ubuntu 24.04 / ROS 2 Jazzy**.

[Installation & Build](#installation) · [Quick Start](#quick-start-without-hardware) · [Bringup Guide](src/allegro_hand_v6/allegro_hand_v6_bringup/README.md) · [UI Manual](src/allegro_hand_v6_ui/README.md)

> [!IMPORTANT]
> **F model:** start bringup and open the dashboard directly. Camera discovery and finger mapping are not required.
> For the V6 V, see the [V-model repository](https://github.com/Wonikrobotics-git/allegro_hand_ros2_v6_v).

---

## ✨ Key Features

- **20-joint control** — ROS 2 hardware integration and controller bringup for left and right hands.
- **Live dashboard** — view the 3D hand, joint feedback, and available pressure readings in one Qt application.
- **Pose tools** — save joint poses, send selected poses, and run timed sequences from the desktop GUI.
- **Direct F-model setup** — launch the hand and dashboard without camera mapping.
- **Browser monitoring** — inspect joint plots and compatible sensor streams from a browser.
- **CSV recording** — record joint positions, measured currents, and pressure values from the configured 500 Hz control loop.
- **Mock preview** — explore the hand model and joint-state integration before connecting hardware.

<a id="models"></a>

## 🧩 F and V Models

Both repositories share the same control software and model assets. Select the model that matches your hand:

| Model | Bringup argument | Camera mapping |
|---|---|---|
| **V6 F** · this repository | `model:=v6_f` | Not required. |
| [V6 V](https://github.com/Wonikrobotics-git/allegro_hand_ros2_v6_v) | `model:=v6_v` | Required for initial assignment of camera images to fingers. |

Examples below use the **F model, right hand**. For a left hand, use `hand:=left` consistently in bringup, the dashboard, and camera mapping where applicable.

---

<a id="installation"></a>

## 🚀 Installation & Build

Install **ROS 2 Jazzy Desktop on Ubuntu 24.04**, then build the base packages and UI together using the steps below.

> [!NOTE]
> This repository contains the complete base and UI workspace. All guides use the same installation; source its `install/setup.bash` in each terminal.

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

### 4. Load the Environment

Use the following in **every terminal** that runs bringup or a UI application:

```bash
source /opt/ros/jazzy/setup.bash
source ~/allegro_hand_ros2_v6_f/install/setup.bash
```

---

<a id="quick-start-without-hardware"></a>

## 🧪 Quick Start Without Hardware

**Terminal 1 — Start mock bringup**

```bash
ros2 launch allegro_hand_v6_bringup allegro_hand.launch.py \
  model:=v6_f hand:=right \
  ros2_control_hardware_type:=mock_components gui:=false
```

**Terminal 2 — Open the dashboard**

```bash
ros2 launch allegro_hand_v6_dashboard dashboard.launch.py hand:=right
```

The dashboard displays the robot model and joint feedback from bringup. Its `hand` argument selects the 2D hand layout; the 3D model comes from bringup. Physical pressure readings require a connected hand. **No camera mapping is needed for the F model.**

---

<a id="run-with-a-physical-hand"></a>

## 🦾 Run with a Physical Hand

Follow the [hardware setup guide](src/allegro_hand_v6/allegro_hand_v6_bringup/README.md#hardware-setup) to connect power and Ethernet and configure the PC's network address. Stop mock bringup before starting the hardware driver.

**Terminal 1 — Start the hardware driver**

Use the actual IP address and port of your hand. The position controller enables torque automatically; no GUI Servo ON action is needed. This command activates the physical hardware interface:

```bash
ros2 launch allegro_hand_v6_bringup allegro_hand.launch.py \
  model:=v6_f hand:=right \
  ros2_control_hardware_type:=hardware \
  io_interface_descriptor:=modbus_tcp:192.168.1.100:502 \
  gui:=false
```

In terminal 2, launch the dashboard as shown above. Sending poses from the pose GUI commands real joints.

For an English walkthrough from feedback checks to a first mock movement and a pose sequence, see [First pose and sequence](src/allegro_hand_v6_ui/allegro_hand_v6_gui/README.md#first-pose-and-sequence). For browser plots, see [UI usage](src/allegro_hand_v6_ui/README.md#optional-browser-monitor). Use the same hand setting and `ROS_DOMAIN_ID` across applications.

<a id="ui-applications"></a>

## 🖥️ UI Applications

| Application | What it does | Usage guide |
|---|---|---|
| **Dashboard** | Displays the live 3D hand and available sensor feedback. | [Dashboard manual](src/allegro_hand_v6_ui/allegro_hand_v6_dashboard/README.md) |
| **Pose Control GUI** | Saves and sends 20-joint poses and runs pose sequences. | [Pose control](src/allegro_hand_v6_ui/README.md#optional-pose-control-gui) |
| **Browser Monitor** | Plots joint data and compatible sensor inputs in a browser. | [Browser monitoring](src/allegro_hand_v6_ui/README.md#optional-browser-monitor) |

Start the base controller before using these applications. The [UI manual](src/allegro_hand_v6_ui/README.md) describes topic requirements and configuration.

---

<a id="optional-csv-recording"></a>

## 📊 CSV Data Recording

Add the following arguments to the physical bringup command:

```text
csv_logging:=true csv_directory:=./data_logs
```

Recording is disabled by default. CSV files contain timestamps, status counters, 20 joint positions, 20 measured currents, and 18 pressure values. The configured control rate is **500 Hz**; actual acquisition rate depends on the device, transport, and PC. See the [CSV recording guide](src/allegro_hand_v6/allegro_hand_v6_bringup/CSV_LOGGING.md) for details.

---

<a id="repository-layout"></a>

## 📦 Repository Layout

```text
src/
├── allegro_hand_common/   # Shared messages, I/O, utilities, and controllers
├── allegro_hand_v6/       # Bringup, robot descriptions, and hardware driver
└── allegro_hand_v6_ui/    # Dashboard, pose GUI, browser monitor, and camera tools
```

The camera tools are retained as part of the shared software suite. F-model operation does not require camera discovery or finger mapping.

<a id="documentation"></a>

## 📚 Documentation

| Guide | Contents |
|---|---|
| [Bringup and hardware setup](src/allegro_hand_v6/allegro_hand_v6_bringup/README.md) | Ethernet setup, mock and physical operation, launch arguments, and controller checks. |
| [UI usage](src/allegro_hand_v6_ui/README.md) | Dashboard, pose control GUI, browser monitor, and their ROS interfaces. |
| [Dashboard manual](src/allegro_hand_v6_ui/allegro_hand_v6_dashboard/README.md) | Visualization, pressure display, layouts, and dashboard settings. |
| [CSV recording guide](src/allegro_hand_v6/allegro_hand_v6_bringup/CSV_LOGGING.md) | Recording options, data columns, and rate checks. |

Version **1.0.0** uses semantic versioning and Git tags such as `v1.0.0`.

---

<a id="contributors"></a>

## 🤝 Contributors

Maintained and released by [Wonikrobotics-official](https://github.com/Wonikrobotics-official).

- [shy0914](https://github.com/shy0914) — development and documentation contributions.

<a id="license"></a>

## 📄 License

Licensed under the [MIT License](LICENSE), except where an individual file or package specifies another license. Existing copyright and license notices are retained, including BSD and Apache-2.0 notices.

Copyright (c) 2026 WonikRobotics_official.

See also the dashboard's [third-party notices](src/allegro_hand_v6_ui/allegro_hand_v6_dashboard/THIRD_PARTY_NOTICES.md) for its external libraries.

<a id="useful-links"></a>

## 🔗 Useful Links

- [Allegro Hand website](https://www.allegrohand.com/)
- [F-model repository](https://github.com/Wonikrobotics-git/allegro_hand_ros2_v6_f)
- [V-model repository](https://github.com/Wonikrobotics-git/allegro_hand_ros2_v6_v)
