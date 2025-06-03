
### Raspberry Pi Camera Installation on Raspberry Pi 5 / Ubuntu 24.04 Noble /ROS 2 Jazzy

#### **<u>Purpose</u>**

This document is based on discussions on the HBRobotics Forum Dec 17-18,
2024, with much appreciation for all those contributing:
<https://groups.google.com/g/hbrobotics/c/4VITfijo2cM/m/80LidlKAAgAJ> .
An additional reference with basic and python bindings testing is :
<https://github.com/slgrobotics/robots_bringup/blob/main/Docs/Sensors/Camera.md>

Here is a description of  the installation, configuration and running of a
**Raspberry Pi Camera Model 2 or 2.1** (imx219) connected to a
**Raspberry Pi 5/4GB** configured with **Ubuntu 24.04 OS** and **ROS2
Jazzy**. Future updates could enable the use of V3 (imx708) or
3<sup>rd</sup> party cameras, e.g. Waveshare IMX219 79.3 0r 120 FOV.

The principle installed package a ROS 2 “**camera_ros**” package :
<https://github.com/christianrauch/camera_ros> that publishes the camera
video on a ros topic.

### 1. Installation and configuration Steps

#### 1.1 Install Ubuntu OS – Ubuntu 24.04 on Raspberry Pi (64 Bit)**  
https://ubuntu.com/download/raspberry-pi  
Use the Raspberry Pi **rpi-imager**  
Select Raspberry Pi 5  
Choose OS, Select Other General Purpose OS\>Select Ubuntu>Ubuntu Desktop 24.04.2 LTS (64 Bit)  
Insert suitable SD Card in the USB Reader Adapter>Choose Storage\>Next  
Check Kernal, of which the current latest is 5.15  
\$ uname -a  
Linux rp4-ub24h-mt 5.15.0-1073-raspi#76-Ubuntu SMP PREEMPT Wed Feb 19
10:39:24 UTC 2025 aarch64 aarch64 aarch64 GNU/Linux  

#### 1.2 Install Camera Cable into Camera and MIPI Connector**
Note the Cable has a smaller 22x0.5 mm end for the RPi and 15x1 mm end
for the camera.  
On Camera end of cable: insert cable with Blue or Orange side facing
away from the USB & Ethernet connectors.

Preferably to use the "cam1" connector as that is the Defefault and has more performance parameters.   
On RPI End insert cable with Blue or Orange side facing USB Connectors

#### 1.3 Add User ( in this case ubuntu), to video group**  
\$ sudo usermod -a -G vídeo ubuntu

#### 1.4 Install ros-jazzy-desktop, build packages and configure workspace**
<https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html>
<https://docs.ros.org/en/jazzy/Tutorials/Beginner-Client-Libraries/Colcon-Tutorial.html>

#### 1.5 Install Dependencies  
v4l2 Utilities to manage a camera, a ROS 2 package that publishes camera
output as a topic, Raspberry Pi configuration utility, and a ROS 2
package that used to subscribe to and publish images. It provides
transparent support for transporting images in low-bandwidth compressed
formats.

Install raspi-config, v4l-utils. ros-jazzy-image-transport-plugins
\$ sudo apt-get install ros-jazzy-image-transport-plugins
v4l-utils

While some camera installation procedure incldue installing the Raspberry Pi raspi-config, it is **NOT** recommended be used to set the camera parameers due reports of breaking the SD Card. It is useful for setting other Interface Paramnets, such is 12C if that is used.


- raspi-config: A tool for configuring camera device connection on
  Raspberry Pi.
- ros-jazzy-image-transport-plugins
- v4l-utils: A utility that assists with connection.

**raspi-config**  
<https://www.raspberrypi.com/documentation/computers/configuration.html>

It helps you configure your Raspberry Pi. Changes to raspi-config will
modify [/boot/firmware/config.txt](https://www.raspberrypi.com/documentation/computers/config_txt.html#what-is-config-txt) and
other configuration files. This procedure describes using this package
directly or editing the config.txt file rather than.

Run \$ sudo apt install raspi-config 
  
#### 1.6 Configure the following parameters on the Raspberry Pi 5 SD Card###

**On the Raspberry Pi /boot/firmware/config.sys file**

Manually edit by: **\$ sudo nano
/boot/firmware/config.sys**

\# Autoload overlays for any recognized cameras or displays that are
\# attached to the CSI/DSI ports. Please note this is for libcamera support as used in these procedures,
 **not** for the legacy camera stack  
camera\_auto\_detect=0  
\# Comment out legacy camera stack  
\# with v4l2 Package  
\#camera\_auto\_detect=1  
start_x=1  
display\_auto\_detect=1  

Put this line at the very end of the config.txt file **NOT** in the middle along with the other camera paramters.

\# Camera Models  
\[all]  
\# Model 2 on Port Cam 1  
dtoverlay=imx219, cam0  
\ #Model 3  
dtoverlay= Imx708, cam1  

**Reboot the Raspberry Pi**

Running v4l2 will list the “legacy” supported camera  
rp1-cfe (platform:1f00110000.csi):  
/dev/video0

**v4l2 Utilities examples**  
Displays all available information for connected Camera Devices  
\$ v4l2 --all  
Shows the device name of a connected Raspberry Pi Camera as device
/dev/video0  
\$ v4l2-ctl –list devices  
rp1-cfe (platform:1f00110000.csi):  
/dev/video0

**image_transport** : github: <https://wiki.ros.org/image_transport>  
These plugin packages may be described in a future document revision.

**Note:**

**ros-jazzy-v4l2-camera is a** package that may be installed on the
**Raspberry Pi** to publish camera output as a topic and should **NOT**
be installed on the Raspberry Pi in this procedure

To verify its status: **\$ sudo apt purge ros-jazzy-v4l2-camera-package**,
should confirm this status.
### 1.7 Install Libcamera Package  
Quoted from [slgrobotics/robots_bringup](https://github.com/slgrobotics/robots_bringup/blob/main/Docs/Sensors/Camera.md),  Marco Walther ported the latest Raspberry Pi **libcamera** changes here:  
GW PPA Repository for Raspberry Pi 5 Ubuntu 24.04 Noble arm64 libcamera
Packages  
<https://launchpad.net/~marco-sonic/+archive/ubuntu/rasppios>  

**Included Built Packages**

- **gstreamer1.0-libcamera** complex camera support library (GStreamer
  plugin)
- **libcamera-dev** complex camera support library (development files)
- **libcamera-doc** complex camera support library (documentation)
- **libcamera-ipa** complex camera support library (IPA modules)
- **libcamera-tools** complex camera support library (tools)
- **libcamera-v4l2** complex camera support library (V4L2 module)
- **libcamera0.5** complex camera support library
- **python3-libcamera** complex camera support library (Python bindings)

**Important Warning:** 
- The contents of [Marco's Personal Package Archives](https://launchpad.net/~marco-sonic/+archive/ubuntu/rasppios) are _not checked or monitored_. You install software from them **at your own risk**. 

\$ sudo add-apt-repository ppa:marco\-sonic/rasppios  
\$ sudo apt update && \$ sudo apt upgrade  

On my machine previously installed "libcamera" package was automatically upgraded with Marco's binary.

\$ sudo apt install libcamera-tools rpicam-apps-lite python3-picamera2

Make sure you are a member of group "_video_", for example my "_ros_" account already is:

ubuntu@rp5-ub24j-mb:~\$ sudo adduser ros video  
[sudo] password for ros:  
info: The user \`ros' is already a member of `video'.  

**Reboot**

After doing this, check the installation with the **dpkg** command  
\$ sudo dpkg -l \|grep libcamera

The usual \$ sudo apt update && sudo apt upgrade will update the
binaries to a newer version.

**Note:**

If during a system update or after running rosdep Marco's packages are replaced, you can fix that easily, for example: following the instructions near the end of: 
<https://github.com/slgrobotics/robots_bringup/blob/main/Docs/Sensors/Camera.md>  

and run  
ubuntu@rp5\-ub24j\-mb:~/cam\_ws \$ sudo apt remove libcamera0.5 ros-jazzy\-libcamera  

### A possible future release is” rpicam-apps”

#### 1.8 Install “camera_ros” package from Source, that publishes camera output as a topic

A helpful background reference for a similar camera package installation
is:
<https://emanual.robotis.com/docs/en/platform/turtlebot3/sbc_setup/#sbc-setup>

**Developed and maintained by:**  
**<https://github.com/christianrauch/camera_ros>**
  
\$ mkdir -p camera\_ws/src  
\$ cd camera\_ws  
\$ git clone git clone  
<https://github.com/christianrauch/camera_ros.git>  
\$ \# resolve binary dependencies and build workspace  
\$ source /opt/ros/\$ROS\_DISTRO/setup.bash  
\$ cd ~/camera\_ws/  
\$ rosdep install -y --from-paths src --ignore-src --rosdistro
\$ROS_DISTRO --skip-keys=libcamera  
#### Delete ros-jazzy-libcamera, \$ sudo remove ros-jazzy-ros-camera  
\$ colcon build --event-handlers=console\_direct+ 
\$ . install/setup.bash

**OR to permanently configure this package to run from anywhere**  
With \$ nano edit .bashrc, and add the line:  
\$ source /home/ubuntu/camera_ws/install/setup.bash  

Now, any Terminal that opens will source this package.  
**In a Terminal**,  
\$ ros2 run camera_ros camera_node -–ros-args -p camera:=0 -p
role:=viewfinder  

**In a 2<sup>nd</sup> Terminal,**  
\$ ros2 run rqt_image_view rqt_image_view /camera/image_raw

#### Node, topic and param list are as follows:**

**ubuntu@rp5-ub24j-mb**:**~**\$ ros2 node list  
/camera

**ubuntu@rp5-ub24j-mb**:**~**\$ ros2 topic list  
/camera/camera_info  
/camera/image_raw  
/camera/image_raw/compressed  
/parameter_events  
/rosout

**ubuntu@rp5-ub24j-mb**:**~**\$ ros2 param dump /camera  
/camera:  
ros\_\_parameters:  
  camera: 0  
  format: ''  
  height: 0  
  jpeg_quality: 95  
  qos_overrides:  
/parameter\_events:  
 publisher:  
  depth: 1000  
  durability: volatile  
  history: keep_last  
  reliability: reliable  
  role: raw  
  start\_type\_description\_service: true  
  use\_sim\_time: false

If Camera Calibration is not done, on running “camera\_node" an error message may be displayed:  
**Unable to open camera calibration file
/home/ubuntu/.ros/camera\_info/imx219\_\_base_soc_i2c0mux_i2c_1_imx219_10_640x480.yaml** appears
because the calibration file is missing. 

#### You may want to do a [Camera Calibration Procedure](https://docs.nav2.org/tutorials/docs/camera_calibration.html) and [How to Calibrate a Monocular Camera](https://wiki.ros.org/camera_calibration/Tutorials/MonocularCalibration)

After performing the calibration, place the corresponding info file in the specified
folder.  
The camera\_name should be set as imx219\_\_base\_soc\_i2c0mux\_i2c\_1\_imx219\_10\_640x480

#### A sample yaml file

image\_width: 640
image\_height: 480  
camera\_name: imx219\_\_base\_soc\_i2c0mux\_i2c\_1\_imx219\_10\_640x480  
frame\_id: camera  
camera\_matrix:  
  rows: 3  
  cols: 3  
  data: \[322.0704122808738, 0, 199.2680620421962, 0, 320.8673986158544,
155.2533082600705, 0, 0, 1\]  
  distortion\_model: plumb_bob  
  distortion\_coefficients:  
  rows: 1  
  cols: 5  
  data: \[0.1639958233797625, -0.271840030972792, 0.001055841660100477,
-0.00166555973740089, 0\]  
rectification\_matrix:  
  rows: 3  
  cols: 3  
  data: \[1, 0, 0, 0, 1, 0, 0, 0, 1\]  
projection\_matrix:  
  rows: 3  
  cols: 4  
  data: \[329.2483825683594, 0, 198.4101510452074, 0, 0,
329.1044006347656, 155.5057121208347, 0, 0, 0, 1, 0\]

#### For further information (stereo, depth) [Image Processing](https://github.com/ros-perception/image_pipeline)
