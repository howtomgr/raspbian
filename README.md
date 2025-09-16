# Raspberry Pi OS Installation Guide

Raspberry Pi OS (formerly Raspbian) is a free and open-source operating system based on Debian, optimized for the Raspberry Pi hardware. It's the official operating system for all models of Raspberry Pi and serves as a FOSS alternative to proprietary embedded operating systems, providing a full desktop environment and development platform for ARM-based single-board computers.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

### Hardware Requirements
- **Raspberry Pi Models**: Pi 1, 2, 3, 4, 400, Zero, Zero W, Zero 2 W
- **MicroSD Card**: 8GB minimum (16GB+ recommended)
- **Power Supply**: 
  - Pi 4: 5V/3A USB-C
  - Pi 3: 5V/2.5A Micro-USB
  - Other models: 5V/2A Micro-USB
- **Display**: HDMI monitor (or headless setup via SSH)
- **Input**: USB keyboard and mouse (for desktop setup)

### Software Requirements
- **Raspberry Pi Imager**: Official imaging tool
- **Alternative**: balenaEtcher, dd command
- **Computer**: Windows, macOS, or Linux for creating SD card

### Network Requirements
- Ethernet cable (recommended for initial setup)
- Or WiFi credentials for wireless setup
- Internet connection for updates and packages

## 2. Supported Operating Systems

This guide covers installing Raspberry Pi OS on:
- Raspberry Pi 4 Model B (1GB, 2GB, 4GB, 8GB)
- Raspberry Pi 400
- Raspberry Pi 3 Model B/B+
- Raspberry Pi 3 Model A+
- Raspberry Pi 2 Model B
- Raspberry Pi 1 Model B+/A+
- Raspberry Pi Zero W/WH
- Raspberry Pi Zero 2 W
- Compute Module 3/3+/4

Available editions:
- **Raspberry Pi OS with desktop and recommended software** (Full)
- **Raspberry Pi OS with desktop** (Standard)
- **Raspberry Pi OS Lite** (Minimal, no desktop)
- **Raspberry Pi OS (64-bit)** (For Pi 3/4/400/Zero 2)

## 3. Installation

### Method 1: Using Raspberry Pi Imager (Recommended)

#### On Windows

1. **Download Raspberry Pi Imager**:
```powershell
# Download from official website
Start-Process "https://www.raspberrypi.com/software/"

# Or using winget
winget install RaspberryPiFoundation.RaspberryPiImager
```

2. **Run Imager and write image**:
   - Insert MicroSD card
   - Choose OS → Raspberry Pi OS (32/64-bit)
   - Choose Storage → Select your SD card
   - Click Settings gear for advanced options
   - Write the image

#### On macOS

```bash
# Install using Homebrew
brew install --cask raspberry-pi-imager

# Or download DMG from website
curl -L https://downloads.raspberrypi.org/imager/imager_latest.dmg -o rpi-imager.dmg
open rpi-imager.dmg
```

#### On Linux

```bash
# Debian/Ubuntu
sudo apt update
sudo apt install -y rpi-imager

# Arch Linux
yay -S rpi-imager

# Fedora
sudo dnf install -y rpi-imager

# Or use AppImage
wget https://downloads.raspberrypi.org/imager/imager_latest_amd64.AppImage
chmod +x imager_latest_amd64.AppImage
./imager_latest_amd64.AppImage
```

### Method 2: Manual Installation using dd

1. **Download Raspberry Pi OS image**:
```bash
# Latest 32-bit with desktop
wget https://downloads.raspberrypi.org/raspios_armhf/images/raspios_armhf-2024-03-15/2024-03-15-raspios-bookworm-armhf.img.xz

# Latest 64-bit with desktop
wget https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2024-03-15/2024-03-15-raspios-bookworm-arm64.img.xz

# Latest Lite version (no desktop)
wget https://downloads.raspberrypi.org/raspios_lite_armhf/images/raspios_lite_armhf-2024-03-15/2024-03-15-raspios-bookworm-armhf-lite.img.xz

# Extract image
xz -d *.img.xz
```

2. **Write image to SD card**:
```bash
# Find SD card device
lsblk
# or
sudo fdisk -l

# Write image (replace /dev/sdX with your device)
sudo dd if=2024-03-15-raspios-bookworm-armhf.img of=/dev/sdX bs=4M status=progress conv=fsync

# Sync and safely remove
sync
sudo eject /dev/sdX
```

### Pre-boot Configuration (Headless Setup)

1. **Enable SSH**:
```bash
# Mount the boot partition
mkdir -p /mnt/boot
sudo mount /dev/sdX1 /mnt/boot

# Enable SSH
sudo touch /mnt/boot/ssh
```

2. **Configure WiFi**:
```bash
# Create wpa_supplicant.conf
cat << 'EOF' | sudo tee /mnt/boot/wpa_supplicant.conf
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
ap_scan=1
update_config=1

network={
    ssid="YourWiFiSSID"
    psk="YourWiFiPassword"
    key_mgmt=WPA-PSK
}
EOF
```

3. **Configure user (Raspberry Pi OS Bookworm)**:
```bash
# Create userconf.txt with username:encrypted-password
echo 'myusername:$6$salt$encryptedpassword' | sudo tee /mnt/boot/userconf.txt

# Generate encrypted password
openssl passwd -6
```

4. **Unmount**:
```bash
sudo umount /mnt/boot
```

## 4. Configuration

### First Boot Setup

1. **Connect and power on**:
   - Insert SD card into Raspberry Pi
   - Connect display, keyboard, mouse (or use SSH for headless)
   - Connect Ethernet or ensure WiFi is configured
   - Connect power supply

2. **Initial configuration wizard** (Desktop version):
   - Set country, language, timezone
   - Change default password
   - Select WiFi network
   - Update software
   - Restart when prompted

3. **Headless SSH access**:
```bash
# Find Raspberry Pi IP address
# From router DHCP list or:
nmap -sn 192.168.1.0/24

# SSH to Raspberry Pi
ssh pi@raspberrypi.local
# or
ssh pi@192.168.1.XXX

# Default credentials (older versions):
# Username: pi
# Password: raspberry
```

### System Configuration Tool

```bash
# Run configuration tool
sudo raspi-config

# Key settings:
# 1. System Options
#    - Hostname
#    - Password
#    - Boot/Auto Login
# 2. Display Options
#    - Resolution
#    - Underscan
# 3. Interface Options
#    - SSH
#    - VNC
#    - SPI, I2C, Serial
#    - Camera
# 4. Performance Options
#    - Overclock
#    - GPU Memory Split
# 5. Localization Options
#    - Locale
#    - Timezone
#    - Keyboard
# 6. Advanced Options
#    - Expand Filesystem
#    - Network Interface Names
```

### Network Configuration

**Static IP Configuration**:
```bash
# Edit dhcpcd.conf
sudo nano /etc/dhcpcd.conf

# Add for eth0:
interface eth0
static ip_address=192.168.1.100/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1 8.8.8.8

# Add for wlan0:
interface wlan0
static ip_address=192.168.1.101/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1 8.8.8.8

# Restart networking
sudo systemctl restart dhcpcd
```

### Update System

```bash
# Update package lists
sudo apt update

# Upgrade packages
sudo apt full-upgrade -y

# Update firmware
sudo rpi-update

# Clean up
sudo apt autoremove -y
sudo apt autoclean

# Reboot if kernel was updated
sudo reboot
```

## 5. Service Management

### systemd Services

```bash
# Enable/disable services
sudo systemctl enable ssh
sudo systemctl enable vncserver-x11-serviced
sudo systemctl disable bluetooth

# Start/stop services
sudo systemctl start ssh
sudo systemctl stop bluetooth

# Check service status
sudo systemctl status ssh

# List all services
systemctl list-units --type=service
```

### Common Services

1. **SSH Server**:
```bash
# Enable SSH
sudo systemctl enable --now ssh

# Configure SSH
sudo nano /etc/ssh/sshd_config
# Change: PermitRootLogin no
# Add: AllowUsers yourusername

sudo systemctl restart ssh
```

2. **VNC Server**:
```bash
# Enable VNC
sudo raspi-config
# Navigate to: Interface Options → VNC → Yes

# Or via command line
sudo systemctl enable --now vncserver-x11-serviced
```

3. **GPIO Services**:
```bash
# Install GPIO tools
sudo apt install -y python3-gpiozero pigpio

# Enable pigpio daemon
sudo systemctl enable --now pigpiod
```

## 6. Troubleshooting

### Common Issues

1. **No display output**:
```bash
# Edit config.txt on boot partition
# Add or modify:
hdmi_force_hotplug=1
hdmi_drive=2
hdmi_group=2
hdmi_mode=82  # 1080p 60Hz

# For Pi 4 with dual displays:
hdmi_force_hotplug:0=1
hdmi_force_hotplug:1=1
```

2. **SD card corruption**:
```bash
# Check filesystem
sudo fsck /dev/mmcblk0p2

# Enable filesystem check on boot
sudo touch /forcefsck
```

3. **Network issues**:
```bash
# Check interfaces
ip addr show
ifconfig

# Restart networking
sudo systemctl restart dhcpcd
sudo systemctl restart networking

# Check WiFi
iwconfig
sudo iwlist wlan0 scan
```

4. **Boot problems** (Rainbow screen):
   - Insufficient power supply
   - Corrupted boot files
   - Incompatible SD card

### Recovery Mode

1. **Hold Shift during boot** for recovery menu
2. **Edit cmdline.txt** on another computer:
```bash
# Add to end of line:
init=/bin/sh

# Boot and remount root
mount -o remount,rw /
# Fix issues, then:
sync
reboot -f
```

### Debug Information

```bash
# System information
cat /proc/cpuinfo
vcgencmd measure_temp
vcgencmd get_throttled

# Boot messages
dmesg | grep -i error
journalctl -xe

# Hardware info
sudo hwinfo --short
lsusb
lsblk
```

## 7. Security Considerations

### Basic Security

1. **Change default password**:
```bash
passwd
# Or for another user:
sudo passwd username
```

2. **Create new user and disable pi user**:
```bash
# Add new user
sudo adduser myusername
sudo usermod -aG sudo,adm,dialout,cdrom,audio,video,plugdev,games,users,input,netdev,spi,i2c,gpio myusername

# Test sudo access
su - myusername
sudo whoami

# Disable pi user
sudo usermod -L pi
# Or delete:
sudo deluser --remove-home pi
```

3. **SSH security**:
```bash
# Generate SSH keys (on client)
ssh-keygen -t ed25519 -C "pi@raspberrypi"

# Copy to Pi
ssh-copy-id myusername@raspberrypi.local

# Disable password authentication
sudo nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no
sudo systemctl restart ssh
```

### Firewall Configuration

```bash
# Install ufw
sudo apt install -y ufw

# Configure rules
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable firewall
sudo ufw enable
sudo ufw status
```

### Automatic Updates

```bash
# Install unattended-upgrades
sudo apt install -y unattended-upgrades

# Configure
sudo dpkg-reconfigure --priority=low unattended-upgrades

# Edit configuration
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

## 8. Performance Tuning

### Overclocking

```bash
# Edit config.txt
sudo nano /boot/config.txt

# Pi 4 example:
over_voltage=6
arm_freq=2147
gpu_freq=750

# Pi 3B+ example:
arm_freq=1500
gpu_freq=600
over_voltage=4
sdram_freq=560

# Monitor temperature
watch -n 1 vcgencmd measure_temp
```

### Memory Split

```bash
# Configure GPU memory
sudo raspi-config
# Advanced Options → Memory Split

# Or edit directly:
gpu_mem=128  # For headless
gpu_mem=256  # For desktop use
gpu_mem=512  # For GPU-intensive tasks
```

### Boot Optimization

```bash
# Disable unnecessary services
sudo systemctl disable bluetooth
sudo systemctl disable avahi-daemon
sudo systemctl disable triggerhappy
sudo systemctl disable rsyslog

# For headless:
sudo systemctl disable lightdm
sudo systemctl disable plymouth
```

### Storage Optimization

1. **Move to USB boot** (Pi 4):
```bash
# Update bootloader
sudo rpi-eeprom-update -d -a

# Enable USB boot
sudo raspi-config
# Advanced Options → Boot Order → USB Boot
```

2. **Use faster SD cards**: A1/A2 rated cards

3. **Enable TRIM for SSDs**:
```bash
sudo systemctl enable fstrim.timer
```

## 9. Backup and Restore

### Full SD Card Backup

```bash
#!/bin/bash
# backup-pi.sh

# On the Pi
sudo dd if=/dev/mmcblk0 bs=4M | gzip | ssh user@backupserver 'cat > pi-backup-$(date +%Y%m%d).img.gz'

# From another Linux system
sudo dd if=/dev/sdX bs=4M | gzip > pi-backup-$(date +%Y%m%d).img.gz

# Restore
gunzip -c pi-backup.img.gz | sudo dd of=/dev/sdX bs=4M conv=fsync
```

### Incremental Backup with rsync

```bash
#!/bin/bash
# Create backup script
cat << 'EOF' > /home/pi/backup.sh
#!/bin/bash
BACKUP_DIR="/mnt/backup/pi-backup"
EXCLUDE_FILE="/home/pi/.backup-exclude"

# Create exclude file
cat << 'EXCLUDES' > $EXCLUDE_FILE
/proc/*
/sys/*
/dev/*
/tmp/*
/run/*
/mnt/*
/media/*
/lost+found
EXCLUDES

# Run backup
rsync -aAXv --delete \
    --exclude-from=$EXCLUDE_FILE \
    / $BACKUP_DIR/

# Create timestamp
date > $BACKUP_DIR/last-backup.txt
EOF

chmod +x /home/pi/backup.sh

# Add to crontab
(crontab -l 2>/dev/null; echo "0 2 * * * /home/pi/backup.sh") | crontab -
```

### Using rpi-clone

```bash
# Install rpi-clone
git clone https://github.com/billw2/rpi-clone.git
cd rpi-clone
sudo cp rpi-clone /usr/local/sbin

# Clone to another SD card
sudo rpi-clone sda

# Clone with options
sudo rpi-clone -f -p 256M sda
```

## 10. System Requirements

### By Model

| Model | RAM | CPU | Recommended Use |
|-------|-----|-----|-----------------|
| Pi 4 B | 1-8GB | 4x 1.5GHz | Desktop, Server |
| Pi 400 | 4GB | 4x 1.8GHz | Desktop |
| Pi 3 B+ | 1GB | 4x 1.4GHz | IoT, Light Desktop |
| Pi Zero 2 W | 512MB | 4x 1GHz | IoT, Embedded |
| Pi Zero W | 512MB | 1x 1GHz | Basic IoT |

### OS Requirements

- **Full Desktop**: 2GB+ RAM, 16GB+ SD card
- **Desktop**: 1GB+ RAM, 8GB+ SD card  
- **Lite**: 512MB+ RAM, 4GB+ SD card

## 11. Support

### Official Resources
- **Website**: https://www.raspberrypi.com/
- **Documentation**: https://www.raspberrypi.com/documentation/
- **Forums**: https://www.raspberrypi.org/forums/
- **GitHub**: https://github.com/raspberrypi

### Community Support
- **Reddit**: r/raspberry_pi
- **Discord**: Various Raspberry Pi servers
- **Stack Exchange**: raspberrypi.stackexchange.com
- **MagPi Magazine**: Free monthly magazine

## 12. Contributing

### How to Contribute
1. Report issues on GitHub
2. Submit pull requests
3. Write documentation
4. Create tutorials
5. Help in forums

### Development
```bash
# Clone documentation
git clone https://github.com/raspberrypi/documentation.git

# Clone firmware
git clone https://github.com/raspberrypi/firmware.git

# Clone kernel
git clone --depth=1 https://github.com/raspberrypi/linux.git
```

## 13. License

Raspberry Pi OS is based on Debian and includes:
- **Base System**: Various open source licenses
- **Raspberry Pi Firmware**: Custom license
- **Software Packages**: Individual licenses

Key components:
- Linux kernel: GPLv2
- Debian packages: Various DFSG-compliant licenses
- Raspberry Pi tools: BSD-3-Clause

## 14. Acknowledgments

### Credits
- **Raspberry Pi Foundation**: Creating affordable computing
- **Debian Project**: Base operating system
- **Community Contributors**: Thousands of developers
- **Educational Partners**: Schools and makers worldwide

## 15. Version History

### Current Releases
- **Bookworm**: Based on Debian 12 (Current)
- **Bullseye**: Based on Debian 11 (Legacy)
- **Buster**: Based on Debian 10 (Old Stable)

### Release Schedule
- Major releases: Following Debian releases
- Updates: Monthly security and bug fixes
- Kernel updates: Regular updates via rpi-update

## 16. Appendices

### A. GPIO Pinout Reference

```
   3V3  (1) (2)  5V
 GPIO2  (3) (4)  5V
 GPIO3  (5) (6)  GND
 GPIO4  (7) (8)  GPIO14
   GND  (9) (10) GPIO15
GPIO17 (11) (12) GPIO18
GPIO27 (13) (14) GND
GPIO22 (15) (16) GPIO23
   3V3 (17) (18) GPIO24
GPIO10 (19) (20) GND
 GPIO9 (21) (22) GPIO25
GPIO11 (23) (24) GPIO8
   GND (25) (26) GPIO7
 GPIO0 (27) (28) GPIO1
 GPIO5 (29) (30) GND
 GPIO6 (31) (32) GPIO12
GPIO13 (33) (34) GND
GPIO19 (35) (36) GPIO16
GPIO26 (37) (38) GPIO20
   GND (39) (40) GPIO21
```

### B. Common Commands

```bash
# System info
uname -a
cat /etc/os-release
vcgencmd version

# Temperature and voltage
vcgencmd measure_temp
vcgencmd measure_volts
vcgencmd get_throttled

# Memory info
free -h
vcgencmd get_mem arm
vcgencmd get_mem gpu

# GPIO control
gpio readall
pinout

# Camera module
vcgencmd get_camera
raspistill -o test.jpg
raspivid -o test.h264 -t 10000
```

### C. Useful Tools

```bash
# Install development tools
sudo apt install -y \
    build-essential \
    git \
    python3-dev \
    python3-pip \
    python3-gpiozero \
    python3-picamera2 \
    nodejs \
    npm

# Install utility tools
sudo apt install -y \
    htop \
    iotop \
    nmap \
    screen \
    tmux \
    vim \
    curl \
    wget
```

### D. Example Projects

1. **GPIO LED Blink**:
```python
#!/usr/bin/env python3
from gpiozero import LED
from time import sleep

led = LED(17)
while True:
    led.on()
    sleep(1)
    led.off()
    sleep(1)
```

2. **Temperature Monitor**:
```bash
#!/bin/bash
while true; do
    temp=$(vcgencmd measure_temp | cut -d= -f2)
    echo "$(date): $temp"
    sleep 60
done
```

---

For more detailed information and updates, visit https://github.com/howtomgr/raspbian