# APIs

First, make sure your favorite python editor loads the M3 python environment (i.e Launch it **from a terminal**, not the icon shortcut).

## Python

### IDE
I recommend to use [spyder](https://code.google.com/p/spyderlib/) and ipython, there're pretty great (more flexible than eclipse+pydev).
```bash
sudo apt-get install spyder ipython
```

#### Getting started with Meka python

Get the robot class in the m3 python
```python
import m3.humanoid as m3h

meka = m3h.M3Humanoid()
```

Now we have to tell the realtime pc (mega-mob) that we ant to send commands to it
First let’s connect to the m3rt_server

```python
### Connect to the m3rt_server
import m3.rt_proxy as m3p

proxy = m3p.M3RtProxy()

#start the proxy (i.e. connect)
proxy.start()
```
```python
# Now that we are connected to the m3rt_server
# We need to say we want to get status/cmd

# Get status
proxy.subscribe_status(meka)

# Send command
proxy.publish_command(meka)

# (Optional) Get param (allow to modify parameters in realtime)
proxy.publish_param(meka)
## Please note that if you publish parameters, you local version of robot_config will be issued to the server.
## So make sure first that the m3ens folder (that contains robot_config) is the same on the realtime pc, otherwise the remote config will be overridden by your local config.

# Now meka is ready to receive some commands
```

### Command Modes

#### Joint command (position)
Mode **Theta** and **Theta_gc** (with gravity compensation):
```python
## Set the mode Theta with gravity compensation
meka.set_mode_theta_gc('right_arm')
## Set the desired value in degrees for each joint
meka.set_theta_deg('right_arm', [0 0 0 30 0 0 0])
## Set the desired velocity [0;1]
meka.set_slew_rate_proportional('right_arm',[1.0]*meka.get_num_dof('right_arm'))
## Set the desired rigidity of the join [0;1]
meka.set_stiffness('right_arm', [1.0]*meka.get_num_dof('right_arm'))
```

#### Torque command
Mode **Torque** and **Torque_gc**:
```python
## Set the mode torque
meka.set_mode_torque('right_arm')
## Set the desired value in mNm for each joint
meka.set_torque('right_arm', [0 0 0 800 0 0 0]) # 7 values for the 7 joints
## Set the desired rigidity of the join [0;1]
meka.set_stiffness('right_arm', [1.0]*meka.get_num_dof('right_arm'))
```

#### ***EXPERIMENTAL*** Velocity command
Mode **thetadot** and **thetadot_gc**:
```python
meka.set_mode_thetadot_gc('right_arm')
## Set the desired value in degrees/s for each joint
meka.set_thetadot('right_arm', [0 0 0 0 0 0 0]) # Stay immobile
## Set the desired rigidity of the join [0;1]
meka.set_stiffness('right_arm', [1.0]*meka.get_num_dof('right_arm'))
```
#### Send/Receive commands 
```python
# Send command/ receive status for all the subscribed components
proxy.step()
```
#### Stop the connection
```python
# Stop the connection
proxy.stop()
```

## Moveit! Python API

http://moveit.ros.org/wiki/MoveIt_Commander 
