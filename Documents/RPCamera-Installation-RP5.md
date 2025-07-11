
### 1. Raspberry Pi Camera Installation on Raspberry Pi 5 / Ubuntu 24.04 Noble /ROS 2 Jazzy

#### 1.1. **<u>Purpose</u>**

This document is based on discussions on the HBRobotics Forum Dec 17-18,
2024, with much appreciation for all those contributing:
<https://groups.google.com/g/hbrobotics/c/4VITfijo2cM/m/80LidlKAAgAJ> .
An additional reference with basic and python bindings testing is :
<https://github.com/slgrobotics/robots_bringup/blob/main/Docs/Sensors/Camera.md>

Here is a description of  the installation, configuration and running of a
**Raspberry Pi Camera Model V1, V2, V2.1, or V3** , CV5647(v1), imx219(v2), or imx_708 (V3) MIPI cable connected  to a
**Raspberry Pi 5/4GB** configured with **Ubuntu 24.04 OS** and **ROS2
Jazzy**. 
3<sup>rd</sup> party cameras, e.g. Waveshare IMX219 79.3 or 120 deg FOV may also function depending on their compatibility.

While this document was intended to apply to a Raspberry Pi 5, it also should work with a Raspberry Pi 4.

The principle installed package a ROS 2 “**camera_ros**” package :
<https://github.com/christianrauch/camera_ros> that publishes the camera
video data on ros topics. The necessary libraries are installed from binaries.

### 2. Installation and configuration Steps

#### 2.1. Install Ubuntu OS – Ubuntu 24.04 on Raspberry Pi (64 Bit)
  
https://ubuntu.com/download/raspberry-pi  
  
Use the Raspberry Pi **rpi-imager**  
Select Raspberry Pi 5  
Choose OS, Select Other General Purpose OS\>Select Ubuntu>Ubuntu Desktop 24.04.2 LTS (64 Bit)  
Insert suitable SD Card in the USB Reader Adapter>Choose Storage\>Next  
Check Kernal, of which the current latest is 5.15  
\$ uname -a  
Linux rp4-ub24h-mt 5.15.0-1073-raspi#76-Ubuntu SMP PREEMPT Wed Feb 19
10:39:24 UTC 2025 aarch64 aarch64 aarch64 GNU/Linux  

#### 2.2. Install Camera Cable into Camera and MIPI Connector**
Note the Raspberry Pi 5 Canera Cable has a smaller 22x0.5 mm end for the RPi5 connector and 15x1 mm end for the camera connector.  
On Camera end of cable: insert cable with Blue or Orange side facing the camera front.
 
On RPI5 End insert cable with Blue or Orange side facing USB Connectors. Preferably to use the "cam1" connector as that is the Default and addresses more performance parameters.  

#### 2.3.Add User ( in this case ubuntu), to video group**  
\$ sudo usermod -a -G vídeo ubuntu

#### 2.4. Install ros-jazzy-desktop, build packages and configure workspace**
<https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html>
<https://docs.ros.org/en/jazzy/Tutorials/Beginner-Client-Libraries/Colcon-Tutorial.html>

To update rosdep, if not already done:  
\$  sudo rosdep init  
\$  rosdep update

#### 2.5. Install Utilities  
v4l2 Utilities functions are to manage a camera, a ROS 2 package that publishes camera
output as a topic, Raspberry Pi configuration utility, and a ROS 2
package that used to subscribe to and publish images. It provides
transparent support for transporting images in low-bandwidth compressed
formats.

- v4l-utils: A camera mangement utility.
- ros-jazzy-image-transport-plugins
- raspi-config: A tool for configuring camera device connection on
  Raspberry Pi.

Install raspi-config, v4l-utils, v4l2tools, ros-jazzy-image-transport-plugins  
\$ sudo apt install ros-jazzy-image-transport-plugins v4l-utils  


While some camera installation procedure include installing the Raspberry Pi raspi-config, it is **NOT** recommended be used to set the camera parameers due reports of breaking the SD Card. It is useful for setting other Interface Parameters, such is 12C if that is used.

**raspi-config**  
<https://www.raspberrypi.com/documentation/computers/configuration.html>

It helps you configure your Raspberry Pi. Changes to raspi-config will
modify [/boot/firmware/config.txt](https://www.raspberrypi.com/documentation/computers/config_txt.html#what-is-config-txt) and
other configuration files. This procedure describes using this package
directly or editing the config.txt file rather than.

Run \$ sudo apt install raspi-config 

**Configure a Fix for DMA Heap Permissions that occurs on some installations**  
- Create a udev rule: This addresses a common issue where the dma\_heap device, essential for the camera to function, only has root permissions by default.  
- Create the file: /etc/udev/rules.d/raspberrypi.rules.  < file can have any name, I used picam.rules.  
- Add this line: SUBSYSTEM=="dma_heap", GROUP="video", MODE="0660".  
- Reload udev rules: Either reboot or run udevadm control --reload-rules && udevadm trigger.  
- Ensure user is in the 'video' group: Make sure the user you are running libcamera-hello as is part of the video group. 
  
#### 2.6. Configure the following parameters on the Raspberry Pi 5 SD Card

**On the Raspberry Pi /boot/firmware/config.txt file**

Manually edit by: \$ sudo nano
/boot/firmware/config.txt

***/boot/firmware/config.txt***  

This is the DEFAULT config.txt file. Much of these lines can be ignored and left as they are, but   
notably revise "camera_auto_detect=1 to "camera_auto_detect=0" where necessary e.g. For Raspberry Pi 5 ov5647 V1, imx219 V2, imx_708 V3 Cameras.  
\___________________________________________________________ 

[all]  
arm_64bit=1  
kernel=vmlinuz  
cmdline=cmdline.txt  
dtoverlay=disable-bt   
initramfs initrd.img follow kernel  
  
\# Enable the audio output, I2C and SPI interfaces on the GPIO header. As these  
\# parameters related to the base device-tree they must appear before any  
\# other dtoverlay\=specification  
dtparam=audio=on  
dtparam=i2c_arm=on  
dtparam=spi=on  
  
\# Comment out the following line if the edges of the desktop appear outside  
\# the edges of your display  
disable\_overscan=1  
  
\# If you have issues with audio, you may try uncommenting the following line  
\#which forces the HDMI output into HDMI mode instead of DVI (which doesn't  
\# support audio output)  
\#hdmi\_drive=2  
  
\# Enable the KMS ("full" KMS) graphics overlay, leaving GPU memory as the  
\# default (the kernel is in control of graphics memory with full KMS)  
dtoverlay\=vc4-kms-v3d  
disable\_fw\_kms\_setup\=1  
  
\# Autoload overlays for any recognized cameras or displays that are attached  
\# to the CSI/DSI ports. Please note this is for libcamera support, not for  
\# the legacy camera stack  
camera\_auto\_detect\=1  
start_x=1  
display\_auto\_detect\=1  
   
\# Config settings specific to arm64  
dtoverlay\=dwc2  

[pi4]  
max_framebuffers=2  
arm_boost=1  
  
[pi3+]  
\# Use a smaller contiguous memory area, specifically on the 3A+ to avoid an  
\# OOM oops on boot. The 3B+ is also affected by this section, but it shouldn't  
\# cause any issues on that board  
dtoverlay\=vc4-kms-v3d,cma-128  

[pi02]  
\# The Zero 2W is another 512MB board which is occasionally affected by the same  
\# OOM oops on boot.  
dtoverlay\=vc4-kms-v3d,cma-128  
  
[cm4]  
\# Enable the USB2 outputs on the IO board (assuming your CM4 is plugged into  
\# such a board)  
dtoverlay\=dwc2,dr_mode=host  
  
[all]  

\___________________________________________________________   
Extra Notes:
\# Autoload overlays for any recognized cameras or displays that are
\# attached to the CSI/DSI ports. Please note this is for libcamera support as used in these procedures,
 **not** for the legacy camera stack  
camera\_auto\_detect\=0  
\# Comment out legacy camera stack  
\# with v4l2 Package  
\#camera\_auto\_detect\=1  
start_x=1  
display\_auto\_detect\=1  

Put the following lines at the very end of the config.txt file **NOT** in the middle along with the other camera paramters.

\# Camera Models on RPi 4  
\[all]  
dtoverlay\=imx219  

OR  
\# dtoverlay\=imx708
OR  

\# Camera Models on RPi 5  
\[all]  
\# Model 2 on Port Cam 1  
dtoverlay\=imx219, cam0  

OR  
\# Model 3  
dtoverlay\= imx708, cam1  

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

**ros-jazzy-v4l2-camera** is a package that may be installed on the
**Raspberry Pi** to publish camera output as a topic and should **NOT**
be installed on the Raspberry Pi in this procedure

To verify its status: **\$ sudo apt purge ros-jazzy-v4l2-camera-package**,
should confirm this status.
### 3. Install Libcamera Package  
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
ubuntu@rp5\-ub24j\-mb:~/cam\_ws \$ sudo apt ros-jazzy\-libcamera  

### 4. ROS 2 "camera_ros" Package

#### 4.1 Install “camera_ros” package from Source, that publishes camera output as a topic

A helpful background reference for a similar camera package installation
is:
<https://emanual.robotis.com/docs/en/platform/turtlebot3/sbc_setup/#sbc-setup>

**Developed and maintained by:**  
**<https://github.com/christianrauch/camera_ros>**

\$ mkdir -p camera\_ws/src  
\$ cd camera\_ws  
\$ git clone <https://github.com/christianrauch/camera_ros.git>  
\# resolve binary dependencies and build workspace  
\$ source /opt/ros/\$ROS\_DISTRO/setup.bash  
\$ cd ~/camera\_ws/  
\$ rosdep install -y --from-paths src --ignore-src --rosdistro
\$ROS_DISTRO --skip-keys=libcamera  
#### 4.2. Delete ros-jazzy-libcamera, \$ sudo remove ros-jazzy-ros-camera  
\$ colcon build --event-handlers=console\_direct+  
\$ . install/setup.bash

**OR to permanently configure this package to run from anywhere**  
With \$ nano edit .bashrc, and add the line:  
\$ source /home/ubuntu/camera\_ws/install/setup.bash  
\$ . .bashrc  
To Check instllation
\$ ros2 pkg list|grep camera, should list
camera_calibration _parsers 
camera_info_manager
camera_ros

Now, any Terminal that opens will source this package.  
**In a Terminal**,  
\$ ros2 run camera\_ros camera_node -–ros-args -p camera:=0 -p
role:=viewfinder  

**In a 2<sup>nd</sup> Terminal,**  
\$ ros2 run rqt\_image\_view rqt\_image\_view /camera/image_raw

#### 4.3. Node, topic and param list are as follows:**

**ubuntu@rp5-ub24j-mb**:**~**\$ ros2 node list  
/camera

**ubuntu@rp5-ub24j-mb**:**~**\$ ros2 topic list  
/camera/camera\_info  
/camera/image\_raw  
/camera/image\_raw/compressed  
/parameter\_events  
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
/home/ubuntu/.ros/camera\_info/imx219\_\_base\_soc\_i2c0mux\_i2c\_1\_imx219\_10\_640x480.yaml or  
/home/ubuntu/.ros/camera\_info/ov5647\_\_base_soc\_i2c0mux\_i2c\_1\_ov5647\_36\_800x600.yaml**  
appears depending on the Camera Model, because the calibration file is missing from its expected location at  
"**/home/\<username>/.ros/**" 

#### 4.4. You may want to do a [Camera Calibration Procedure](https://docs.nav2.org/tutorials/docs/camera_calibration.html) described in the **NAV2** General Tutorials. 

After performing and sucessfullly completing the calibration, an **xvf** zip file **calibration.tar.gz** is placed in the RasPi5  
/tmp/ directory. Unzipping this file will generate **ost.yaml**, **ost.txt** files, and a set of calibration images in this directory. Move or copy the **ost.yaml** into the camera\_info folder location stated in the error messsage, renamimg it to the camera_name defined in the error message. The camera\_name should be set in the stated full file name: e.g. imx219\_\_base\_soc\_i2c0mux\_i2c\_1\_imx219\_10\_640x480.yaml

#### 4.5. A sample yaml file

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

#### 4.6. For further information (stereo, depth) [Image Processing](https://github.com/ros-perception/image_pipeline)
