# SteamDeck-ROS2-Robot-Controller

This guide explains how to turn a Steam Deck into a mobile-robot development workstation. It covers booting Ubuntu 24.04 from a microSD card, installing ROS 2 Jazzy, and configuring a productive workflow for building and testing robots.

## Why use the Steam Deck?

- Powerful AMD APU with built-in screen, controls, Wi-Fi, and Bluetooth.
- Runs standard x86_64 Linux, so ROS 2 and robotics tools work natively.
- microSD slot allows a portable, swap-free Linux install without modifying SteamOS.

## Prerequisites

- Steam Deck (LCD or OLED).
- 64 GB+ microSD card (UHS-I or better) and reader.
- USB-C hub or dock with keyboard/mouse (recommended for setup).
- Another computer to flash the microSD image.

## Flash Ubuntu 24.04 to the microSD

1. Download the Ubuntu 24.04 (64-bit) desktop ISO.
2. Insert the microSD card into your flashing machine.
3. Use **Rufus** (Windows), **balenaEtcher**, or `dd` (Linux/macOS) to write the ISO to the microSD.
4. Safely eject the card and insert it into the Steam Deck.

## Boot Ubuntu on the Steam Deck

1. Power off the Steam Deck completely.
2. Hold **Volume Down** and press **Power** to enter the Boot Manager.
3. Choose the microSD (often shown as "EFI SD/MMC" or similar).
4. Follow the Ubuntu installer. When partitioning, install to the microSD to keep SteamOS intact.
5. After installation, return to the Boot Manager and set the microSD as the boot device when you want the development environment.

## Post-install tuning

- Run all updates: `sudo apt update && sudo apt upgrade`.
- Enable performance mode in KDE Power settings or via the performance overlay when compiling.
- (Optional) Install Steam Deck firmware and controller tools: `sudo apt install steam-devices steamlink`.
- Add `amdgpu` firmware if prompted by `dmesg` (typically already included in Ubuntu 24.04).

## Install ROS 2 Jazzy (Ubuntu 24.04)

1. Configure locales:
   ```bash
   sudo apt update
   sudo apt install locales
   sudo locale-gen en_US en_US.UTF-8
   sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
   export LANG=en_US.UTF-8
   ```
2. Add the ROS 2 repository:
   ```bash
   sudo apt install software-properties-common curl gnupg lsb-release
   sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list
   ```
3. Install ROS 2 Jazzy desktop and tools:
   ```bash
   sudo apt update
   sudo apt install ros-jazzy-desktop ros-dev-tools
   ```
4. Source ROS 2 automatically:
   ```bash
   echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
   source ~/.bashrc
   ```

## Create a ROS 2 workspace

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
colcon build --symlink-install
source install/setup.bash
```
Add the workspace overlay to your shell by appending `source ~/ros2_ws/install/setup.bash` to `~/.bashrc`.

## USB, serial, and networking notes

- Add your user to dialout for serial devices: `sudo usermod -aG dialout $USER`.
- For microcontrollers over USB-C hubs, ensure they enumerate as `/dev/ttyACM*` or `/dev/ttyUSB*` and set `udev` rules as needed.
- Wi-Fi 6E and Bluetooth work out of the box; tethering via USB-C or Ethernet docks is ideal for stable robot links.
- If you dual-boot, be mindful of SteamOS taking over the boot priority; use the Boot Manager to re-select the microSD when necessary.

## Development workflow tips

- Use `colcon build --merge-install` for faster incremental builds on the APU.
- VS Code works well; install via `sudo snap install code --classic` or the `.deb` package.
- For simulation, install Gazebo Harmonic (`sudo apt install ros-jazzy-gazebo-*`) and lower graphics settings if frame rates drop.
- Keep a second microSD as a backup image after you finish configuring the system.

## Troubleshooting

- **Black screen after boot:** hold the Steam button to open the performance overlay and verify brightness; try an external monitor if needed.
- **Trackpads not detected:** install the `steam-input` udev rules via `steam-devices` and reboot.
- **Slow builds:** switch the Steam Deck to performance mode, plug into power, and consider using `colcon build --parallel-workers $(nproc)`.

## Next steps

- Clone your robot repositories into `~/ros2_ws/src` and build.
- Set up ROS 2 networking (DDS domain ID and `ROS_LOCALHOST_ONLY=0`) to communicate with robots on the same LAN.
- Create a system image of the microSD once the environment is stable.
