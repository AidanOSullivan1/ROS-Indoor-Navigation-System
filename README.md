# ROS-Indoor-Navigation-System
The research paper associated with this project is in the repository, I would recommend reading this paper to get a clear outline of the project.
This indooor navigation system required the following hardware:

The following text in the README file attempts to summarise the development of the navigation system. Please view the link to the Youtube video at the end of the file to observe the capabilities of the final system.

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

RTABmap:

This package was installed to enable SLAM. The configuration and launch files had to be altered and adapted to work with the existing hardware. This required familiarity to be gained with XML files. Extensive reading was carried out on the papers authored by the creator of this SLAM package ‪Mathieu Labbé, to understand the inner workings of this visual SLAM software. This enabled effective usage of the package and aided with the extensive work carried out to configure the software to the existing hardware and optimise its performance, enabling the computationally heavy software to be ran with the limited power of the on-board Raspberry Pi.

The first step involved mapping the environment seen below. The robot was manually driven around the environment to create the initial map and to develop its visual bag-of-words that would later help with localisation.
 
![image](https://user-images.githubusercontent.com/86829520/151217551-51ed2990-78cd-410c-a529-1bdd8cb1fb44.png)

The obtained map from this process:
![Wheel_odom_map (1)](https://user-images.githubusercontent.com/86829520/151218846-9ce25291-2f7c-425d-9dfa-ccd7ab473cee.png)

Next its localisation ability was tested by placing it in random parts of the mapped area to check how accurately it was capable of positioning itself. A satisfactory performace was achieved with only a 1.2% distance error using fused odometry (Wheel and Visual odometry data):
![robot_in_map (1)](https://user-images.githubusercontent.com/86829520/151371823-42423dc4-1a7e-4ee7-91de-f08102cf2ce3.png)

Navigation:
The navigation challenge built upon what had been developed in the mapping and localisation stages 
of the project. The AMR needed a reliable and accurate map that it could use to set navigation goals 
and plan a collision free path to reach its destination. It was essential that it could robustly localise, 
to compute its point of origin and when transiting that it was constantly aware of its location, in 
order to regularly course correct and supress any navigation errors. This problem was addressed by 
developing a ROS navigation stack, using the move_base package. It combines a global and local 
planner to complete its global navigation assignment. This is achieved by intelligently integrating and 
processing multiple nodes, such as the map, RTABmap localisation, the RGBD senor information, 
odometry and a navigation goal. The output of this process are velocity commands to the Roomba
that guide it to the destination.

The code contained in this github repository deals with the navigation aspect of the AMR.

The global costmap is generated from the static map, and it is used to produce the initial path from the 
base to the destination. It inflates obstacles on the map as seen below (Initial path shown in Green):
![global_path_plan_decision (1)](https://user-images.githubusercontent.com/86829520/151380521-535bb817-2424-4ab6-b701-ddf780b390be.png)

The local costmap represents the environment immediately around the AMR, using the static map,
obstacles, and inflation layer. Creating a map that incorporates the live RGBD camera stream, with 
the nearby section of the static map, imaging its current field of vision and inflating any visible 
obstacles. (New obstacles not present during inital mapping are shown)
![local_path_plan_corner_5](https://user-images.githubusercontent.com/86829520/151381141-d3dfa4b0-458c-4de1-83dd-9b09fc3e3204.png)


Move_base links together the global and local planners, as well as the various 
configuration files that have been discussed. It also integrates the static map and launches RTABmap 
in localisation mode with the relevant mapping database for loop closures. RTABmap inherently 
provides the sensory data from the RGBD sensor. The odometry and base frame id are also set
within the move_base.launch file. Move_base is responsible for instructing the creation of the global 
and local path plans and costmaps, serving as an orchestrator of the entire process.


The navigation and dynamic obstacle avoidance capabilities are shown in this video:
https://www.youtube.com/watch?v=LBoFgoT_Fcs

