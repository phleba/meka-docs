# ControlIt! and Meka

>ControlIt! is a middleware for instantiating whole body operational space controllers for humanoid robots. It provides tasks, constraints, and model-based sensor abstractions for applications, interfaces for connecting to robot hardware, and a mechanism for binding internal parameters to external sources. ControlIt! is available open source under an LGPLv2.1 license.

see [here](http://robotcontrolit.com/)

A short introduction on how to deploy ControlIt! on your machine.

### Prerequisites

You need [ROS](ros.org) and [GAZEBO](http://gazebosim.org/) distribution. This setup is tested with **ROS kinetic** and **Gazebo 7.0.0**

Other packages 
* libserial ~0.6rc2 - [git](https://github.com/crayzeewulf/libserial)
* rbdl 2.3 - [bitbucket](https://bitbucket.org/cfok/rbdl2)
* yaml 0.3 (!!!) - [git](https://github.com/jbeder/yaml-cpp) - build with `cmake -DBUILD_SHARED_LIBS=ON ..`
* muparser 2.2.5 - [git](https://github.com/beltoforion/muparser.git) (or via the debian package `libmuparser-dev`)

### Installation

Set up your local catkin workspace
* source /opt/ros/kinetic/setup.bash
* mkdir -p `my_workspace/src` && cd my_workspace && catkin init
* catkin config -i /your/install/space
* catkin config --install

The following projects are catkin packages, so you can collect them in `my_workspace/src`

**Meka specific**
* m3core - [git](https://github.com/CentralLabFacilities/m3core)
* m3meka - [git](https://github.com/CentralLabFacilities/m3meka)
* rtai 4.1.1 - rtai.org

See [here](https://github.com/CentralLabFacilities/mekabot) for documentation on how to install your local meka setup. 



**ros_shared_memory_interface**
* https://github.com/CentralLabFacilities/ros_shared_memory_interface.git

**Now, check out the ControlIt! repositories**
* https://github.com/CentralLabFacilities/controlit.git
* https://github.com/CentralLabFacilities/controlit_models.git
* https://github.com/CentralLabFacilities/controlit_demos.git
* https://github.com/CentralLabFacilities/controlit_configs.git
* https://github.com/CentralLabFacilities/controlit_dreamer_integration.git

In your catkin workspace you should now be able to built everything via `catkin build`. 

If there are any problems, look [here](https://github.com/liangfok/controlit#introduction) for additional hints.

### Run your first demo!

Look [here](http://robotcontrolit.com/tutorials/dreamer) for official ControlIt! demos. Let's start the first demo...
**Set up your environment**
* `source /your/install/space/setup.bash`

**Set up your local gazebo model database to run the dreamer demos of UTA**
* in your $HOME folder, navigate to `.gazebo/models` (start gazebo for the first time if it does not exist)
* create symlinks to your necessary robot models. examples for this demo to work: `ln -s /your/install/space/share/dreamer_controlit/models/dreamer_controlit/` and `ln -s /your/install/space/share/dreamer/models/dreamer/`

**Run!**
* `roslaunch dreamer_controlit simulate_jpos.launch`

Follow the instructions found in the [official demo deocumentation](http://robotcontrolit.com/tutorials/dreamer/controlit/jpos)

