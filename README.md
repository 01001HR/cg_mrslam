cg_mrslam
=========

A ROS package that implements a multi-robot SLAM system.

Provides a multi-robot graph-based 2D SLAM with any assumption about data association or initial relative positions between robots.  
Handles communication among robots working in an Ad-Hoc network.

This multi-robot SLAM approach is based on the use of *condensed graphs* to share relevant map information among robots.

It is possible to use it also for single-robot SLAM.

Requirements:
-------------
- This code uses the g2o framework for graph optimization  
  Download the **HIERARCHICAL** branch:
  
        $ git clone -b g2o_hierarchical https://github.com/RainerKuemmerle/g2o.git

- [ROS fuerte](http://wiki.ros.org/fuerte/Installation) OR [ROS indigo.](http://wiki.ros.org/indigo/Installation)

The code has been tested on Ubuntu 12.04 and 14.04 (64bits). 

Installation
------------
- Download the source code to your ROS workspace directory
- ROS fuerte:
  - Type `rosmake` in the package directory
- ROS indigo (catkin):
  - Change to branch indigo

            $ git checkout indigo
  - In your catkin workspace 

            $ catkin_make -DCMAKE_BUILD_TYPE=Release

Instructions
------------
There are four main programs for different kind of experiments.  

- **real_mrslam:**
  For running a multi-robot online real experiment. Robots must belong to the same network and their IP addresses configured from 192.168.0.(1.. *nRobots*).  
  By default it reads /scan and /odom ROS topic although they are configurable.
  Each time a robot receives a message from the network, it publishes a message in /ping_msgs to be able to reproduce robot connectivity offline.

- **bag_mrslam:**
  For reproducing bagfiles generated from a multi-robot real experiment. Reads the /ping_msgs topic to reproduce communication among robots.

- **sim_mrslam:**
  For running multi-robot simulation experiments with the Stage simulator. Connectivity among robots is based on their relative distances using the ground-truth positions.

- **srslam:**
  For running a single-robot experiment.

All programs must be run adding the robot namespace "robot_x", where x is the robot identifier (0.. *nRobots* -1).
The default parameters work well in general, anyway for specific program options type:

    $ rosrun cg_mrslam mainProgram --help
  
###Example of use:###
Some simulation bagfiles are provided. In the case of use them, there is no need to launch the Stage simulator.  
Open one terminal and type from the bagfiles directory (make sure you launched `roscore` before):

    $ rosbag play 2robots-hospital.bag

In two different terminals type:

    $ rosrun cg_mrslam sim_mrslam -idRobot 0 -nRobots 2 -scanTopic base_scan -o testmrslam.g2o __ns:=robot_0
    $ rosrun cg_mrslam sim_mrslam -idRobot 1 -nRobots 2 -scanTopic base_scan -o testmrslam.g2o __ns:=robot_1 

During the execution it will create one g2o file for each robot, named robot-x-testmrslam.g2o which can be openned with the **g2o_viewer**.  

Additionally, the graph can be visualized online using RViz.  The map will be drawn with respect the Fixed Frame which can be defined through the command line if it is different from the default one. For the  current example, to visualize the map of robot_0:

    $ rosrun cg_mrslam sim_mrslam -idRobot 0 -nRobots 2 -scanTopic base_scan -fixedFrame /robot_0/odom -o testmrslam.g2o __ns:=robot_0

Then, launch RViz (`rosrun rviz rviz`) and select /robot_0/odom as the Fixed Frame in Global options. The nodes of the graph are published as a PoseArray message on the /robot_0/trajectory topic while their associated laserscans are published as a PoinCloud on the /robot_0/lasermap topic.


Related papers
---------------
Please, cite the following paper when using this code:  

- M.T. Lazaro, L.M. Paz, P.Pinies, J.A. Castellanos and G. Grisetti. Multi-Robot SLAM using Condensed Measurements. IEEE/RSJ International Conference on Intelligent Robots and Systems, Tokyo Big Sight, Japan, Nov 3-8, pp. 1069-1076, 2013.

Acknowledgements
----------------
- Giorgio Grisetti for his guidance in the development of this project during my stay at La Sapienza University of Rome.
- Taigo M. Bonnani from La Sapienza University of Rome for his collaboration in the implementation of the scan matcher.
