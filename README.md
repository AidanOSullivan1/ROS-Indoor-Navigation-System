# ROS-Indoor-Navigation-System
This indooor navigation system required the following hardware:
• IRobot Roomba 600 (publishes wheel odometry)
• Raspberry Pi 4 model B (All SLAM and navigation carried out on-board without the use of external computation power.
• Asus Xtion Pro Live RGBD camera
• I built a custom voltage level shifter interface circuit, to facilitate serial communication between the Roomba and the Raspberry Pi (5V and 3.3V)
• 20 Ah battery to enable several hours of run-time

Choosing the OS to run on the Raspberry Pi:
It was important that the OS supported the packages that would 
be used in this assignment, such as ROS and RTABmap. Initially the pre-installed Raspbian Buster OS
was chosen. However, after a prolonged period of the project was spent trying to unsuccessfully
install packages on this OS, it was realised that this had been a poor choice.
Though it is stated by the developers that they were compatible with Raspbian, many issues and dependencies had not 
been resolved, as there is a relatively small community of users. ROS and most SLAM packages were 
primarily launched on Ubuntu and its strong community regularly publishes updates and quickly 
fixes reported bugs. Instead, the recently released Ubuntu Mate 20.04 edition was burned onto the 
SD card, allowing the necessary packages to be installed with far less problems.
The initial set up requires a mouse, screen, and keyboard to 
be connected to set up SSH, log-in credentials and Wi-Fi connection, so the interface could be 
connected to over a computer, enabling future mobile use. Initially the SSH allowed the computer 
and Raspberry Pi to communicate over a common network through a terminal. This required logging 
into the Raspberry Pi, using the created credentials through the command prompt window to 
initiate connection. When XRDP was installed on the microcontroller, this facilitated Windows 
Remote Desktop connection, providing access to the Ubuntu user interface. The UART port on the 
Raspberry Pi had to be enabled by configuring Ubuntu’s firmware using the following steps:
$ nano /boot/firmware/config.txt
In this file the lines “enable_uart = 1” and “dtoverlay = uart4” were added. This configuration sets up 
the GPIO pins, so pins 8 and 9 were Tx and Rx respectively. Ubuntu assigned the serial port to the 
address /dev/ttyAMA1. This was used to communicate with the Roomba.

Installing ROS:
ROS Noetic was installed as it was a new version. All dependencies had to be installed beforehand. Roomba sensor information then needed to be published on ROS and velocity commands sent to the Roomba from ROS. It was configured to do so and controlled manually at first to gain familiarity with the odometry and other sensory data, as well as how the Roomba was controlled through sending data packets over the serial interface.

RTABmap:This package was installed to enable SLAM. The configuration and launch files had to be altered and adapted to work with the existing hardware. This required familiarity to be gained with XML files. Extensive reading was carried out on the papers authored by the creator of this SLAM package ‪Mathieu Labbé, to understand the inner workings of this visual SLAM package. This enabled effective usage of the package and aided with the extensive work carried out to configure the software to the existing hardware and optimisie its performance.

The first step involved mapping the environment seen below. The robot was manually driven around the environment to create the initial map and to develop its visual bag-of-words that would later help with localisation.
 
![image](https://user-images.githubusercontent.com/86829520/151217551-51ed2990-78cd-410c-a529-1bdd8cb1fb44.png)

The obtained map from this process:
![Wheel_odom_map (1)](https://user-images.githubusercontent.com/86829520/151218846-9ce25291-2f7c-425d-9dfa-ccd7ab473cee.png)

Next its localisation ability was tested by placing it in random parts of the mapped area to check how accurately it was capable of positioning itself. A satisfactory performace was achieved with only a 1.2% distance error using fused odometry:
![robot_in_map (1)](https://user-images.githubusercontent.com/86829520/151371823-42423dc4-1a7e-4ee7-91de-f08102cf2ce3.png)




