# ROS Control and MoveIt! for the Meka robot
===
![Meka moveit](http://googledrive.com/host/0B6zWJ1Gzg1UTVkgtMWJaX1NCdVE/meka_moveit_small.png)

### M3 ROS Control
ROS Control is the (new) standard interface to define controller and control robots. It's a bridge between your custom robot and ROS. You read more about ros_control [here](http://wiki.ros.org/ros_control) for more info.

Installation instructions for M3 ROS Control are available [here](https://github.com/ahoarau/m3ros_control), but it should be already on Meka's realtime PC, in the home.

### Loading
With the new overlay interface, we can add new components by just loading a few environnement variables (essentially the M3_ROBOT variable, that is now a list of controllers locations, and a bunch of others):
```
source ~/m3ros_control/setup.bash
```
### Start the m3rt_server
In the same terminal in which you've loaded the m3ros_control variables, start the server: 
```
m3rt_server_run
```
### Launch the joint trajectory controllers
These standard controllers will create the FollowJointTrajectory action that MoveIt! uses to send trajectories to the robot.
```
roslaunch m3ros_control meka_controllers.launch
```
### MoveIt! Config
Now launch the MoveIt! core:
```
roslaunch meka_moveit meka_moveit.launch
```
And maybe rviz (you need to load the rviz config in meka_moveit/config/*.rviz):
```
roslaunch meka_moveit moveit_rviz.launch
```


> Note: A very good moveit configuration can be found in the [Baxter Cpp repo](https://github.com/davetcoleman/baxter_cpp/).
