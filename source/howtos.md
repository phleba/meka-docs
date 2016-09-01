# How-Tos

## Start up procedure

### Connect to the realtime computer:
```
ssh meka-mob
```
### Start the m3rt_server
```
m3rt_server_run
```
### Calibrate the base
```
m3qa_omnibase_calibrate
```
### Calibrate the z-lift

*attention required!*

Every time the robot is power cycled, the zlift looses it's calibration. If you are planning to use the zlift, launch m3_demo_zlift_z2r1.py to calibrate it.

Please be aware the robot is gonna go **down** so make sure the arm and hands don't touch the floor.

## System introspection

### Check battery levels. 
```
m3_demo_pwr.py 
```
Choose pwr component and press any key except 'o', 'f' or 'q'

## Launch a demo

Demos are located in /usr/local/bin, not all of them are working but at least a few:

```python
# Demo arms
m3_demo_arm_a2r5.py

# Demo hand
m3_demo_hand_h2r5.py

# Demo zlift (you might need to calibrate it, see further up)
m3_demo_zlift_z2r1.py
```

## Tune PID controllers

### M3 Joint Tuning Tool

Tuning the PID controllers is a very easy process using the joint tuning tool. It will help you set gains and send commands to the robot so you can see the modifications in realtime.

>Note: it works by modifying the Param protobuf attached to each component to the realtime server. 

Launch the GUI:
```
m3dev_joint_tuning.py
```
 
### Tuning PID controllers: general information

Check out [PID without a PHD](http://m.eet.com/media/1112634/f-wescot.pdf).

Basically what we need on the robot is fast responsiveness and no overshoot, otherwise it will bump into objects to gasps for example. As velocities are quite noisy, the 'D' parameter of a PID should **not** be set very high. As a rule of thumb, I start by setting 'I' and 'D' to zero, tuning 'P' until it shakes, lower it a bit, then start playing with the 'D' term for a nice behavior (like a slow arrival). Eventually, the 'I' term is set to cancel static error.

>Note: In most cases, the 'D' term is 0 or very low.

**Warning : The robot might shake if the PID gains are set too high. This can cause hardware issues, so keep the red button in hand at all time when tuning the PIDs.**


### Tuning Torque Control PID

Torque is the lowest level from which the other modes derives, so if torque mode works perfectly, then the other modes (theta, thetadot) will work even better.


Set the actuator mode to **torque** (no GC), set a torque cmd (like 1600 mNm), and see how is reacts.


Then the parameters are in the **ctrl_simple** component attached to each actuator, under **pid_torque**. Just set a value and press Enter to send it to the robot. It's as easy as that.

When you're done, you can press ***save*** to save the config file. 

>Note: Arm motors can be quite precise and fast (no overshoot and fast responsiveness), so unless PID torque torque is perfect, don't save it.

>Note: Thanks to overlays, you copy all the **real_robot** content in a different location and overlay the config files so the original configuration of the robot remains unchanged.

### Tuning Position Control PID

Same as above, put the actuator in **theta** mode, then tune the pid in **ctrl_simple**, **pid_theta**.

**Warning**: if you improve the torque mode, then the theta mode will probably become **unstable**, because the previous gains are now **too high** for the robot. Usually you'll have to **lower** them (and that's a nice feature in the end because low gains means more compliance !).


>Note: If you tune it well, you can achieve a **~0.1 degrees accuracy**.

### Tuning Velocity Control PID

PID parameters for the velocity controller are directly in the **joint** component of the actuator.

The names of the parameters are:
```
kqdot_d: 0.0
kqdot_i: 0.0
kqdot_i_limit: 1000.0
kqdot_i_range: 1500.0
kqdot_p: -100.0
```

>Note: as of October 2014, **only right_arm has been calibrated for velocity control**

>Note to dev: these parameters can be switched to the **ctrl_simple component by added a command mode in ctrl_simple.cpp and updating the command mode in joint.cpp


## Meka-mob realtime pc setup from scratch

* Get [Ubuntu 14.04LTS 64bits](http://www.ubuntu.com/download/desktop)

* Get [EtherCAT from Igh](https://github.com/ahoarau/ethercat-drivers)

Remove unnecessary kernel modules that might cause the RTAI system to be unstable:
Open the blacklist conf file: 
```
sudo nano /etc/modprobe.d/blacklist.conf
```
And add all unnecessary modules:
```
blacklist amd76x_edac
blacklist btusb
blacklist ath9k
blacklist ath9k_common
blacklist ath9k_hw
blacklist ath
blacklist mac80211
blacklist cfg80211
blacklist bluetooth
blacklist mac_hid
blacklist intel_powerclamp
blacklist aesni_intel
blacklist ghash_clmulni_intel
blacklist snd_timer
blacklist snd_seq_midi_event
blacklist snd_seq_midi
blacklist snd_seq_device
blacklist snd_seq
blacklist snd_rawmidi
blacklist snd_pcm
blacklist snd_page_alloc
blacklist snd_hwdep
blacklist snd_hda_intel
blacklist snd_hda_codec_realtek
blacklist snd_hda_codec_hdmi
blacklist snd_hda_codec
blacklist snd
blacklist psmouse
blacklist coretemp
blacklist binfmt_misc
blacklist parport_pc
blacklist parport
blacklist serio_raw
blacklist lp
blacklist ppdev
```

* Get RTAI / Mekabot / ROS

You might want to [compile your own RTAI kernel](https://github.com/ahoarau/m3installation/blob/master/rtai-kernel-build.md).
```
Compile with -DETHERCAT=1 -DCMAKE_BUILD_TYPE=Release
```

* Setup the student account 

First create a non-admin account Student with a password, then open the sudoers file:
```
sudo visudo 
```
And add the following:
```
student ALL=(root) NOPASSWD:/sbin/insmod
student ALL=(root) NOPASSWD:/sbin/rmmod
student ALL=(root) NOPASSWD:/usr/local/bin/ethercat
student ALL=(root) NOPASSWD:/usr/bin/service
```
This will allow student to launch the m3rt_server without root password.

## Recording

### M3rec (new): Record in a .txt file

You can use m3rec to record vectors and matrices to a file:

```
m3rec --help # For usage
```

It read the humanoid api and generates all the callable functions as parameters for the utility.

Example:

Record joint angles and velocity to a folder in ~/records, name the output file right_left_th_thdot.txt and record now (otherwise it waits for Enter to be pressed).

```
m3rec --right_arm get_theta_deg get_thetadot_deg --left_arm get_theta_deg get_thetadot_deg -o ~/records -f right_left_th_thdot -r
```

### Rosbag

You need to launch the robot description, the robot state publisher and the joint state publisher:
```
roslaunch meka_description m3ens.launch
```

Then use the rosbag API:
http://wiki.ros.org/rosbag





