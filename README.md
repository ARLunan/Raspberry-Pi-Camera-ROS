## Raspberry-Pi-Camera-ROS
Installation and running a MiPi Raspberry Pi Camera on a Raspberry Pi 5 / 4 GB

The Document was the result of an ongoing dialog on content, posted on the HBRobotics Forum [(https://groups.google.com/g/hbrobotics)) , from notes, Linux Terminal scripts and libraries contributed by Alan Federman, Marco Walther, Sergei Grichine,  Ross Lunan. The necessary libraries are installed from downloaded binaries, from githubs: GW PPA https://launchpad.net/~marco-sonic/+archive/ubuntu/rasppios/+packages, https://github.com/christianrauch/camera_ros, and and https://emanual.robotis.com/docs/en/platform/turtlebot3/sbc_setup/#sbc-setup. 

The February 2026 update describes the use of Debian installed camera_ros package vs Build from Source of previous commits.

In September 2026, it was noticed that the ROS2 has moved to version 0.7 of the libcamera library, while the repos described here are on 0.6 (dated Feb 3). Marco updated his libcamera libraries which need to be installed as described by Sergei (https://github.com/slgrobotics/robots_bringup/blob/main/Docs/Sensors/Camera.md#installation) here.

sudo apt update
sudo apt upgrade
sudo apt purge libcamera-dev libcamera-ipa libcamera-tools libcamera-v4l2 librpicam-app1 python3-libcamera rpicam-apps-core -y
sudo apt autoremove -y
sudo apt install libcamera-dev libcamera-ipa libcamera-tools libcamera-v4l2 librpicam-app1 python3-libcamera rpicam-apps-core
ll /usr/lib/aarch64-linux-gnu/libcamera*
    lrwxrwxrwx 1 root root      21 Sep  2 17:10 /usr/lib/aarch64-linux-gnu/libcamera-base.so -> libcamera-base.so.0.7
    lrwxrwxrwx 1 root root      23 Sep  2 17:10 /usr/lib/aarch64-linux-gnu/libcamera-base.so.0.7 -> libcamera-base.so.0.7.2
    -rw-r--r-- 1 root root  198840 Sep  2 17:10 /usr/lib/aarch64-linux-gnu/libcamera-base.so.0.7.2
    lrwxrwxrwx 1 root root      16 Sep  2 17:10 /usr/lib/aarch64-linux-gnu/libcamera.so -> libcamera.so.0.7
    lrwxrwxrwx 1 root root      18 Sep  2 17:10 /usr/lib/aarch64-linux-gnu/libcamera.so.0.7 -> libcamera.so.0.7.2
    -rw-r--r-- 1 root root 1579248 Sep  2 17:10 /usr/lib/aarch64-linux-gnu/libcamera.so.0.7.2

sudo apt install ros-jazzy-camera-ros
### setting FPS=5 for WiFi:
ros2 run camera_ros camera_node --ros-args -p width:=640 -p height:=480  -p FrameDurationLimits:="[200000,200000]"
