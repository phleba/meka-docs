# Meka robot overview

![Meka robot at Ensta Paristech](http://googledrive.com/host/0B6zWJ1Gzg1UTVkgtMWJaX1NCdVE/meka.jpg)

## Hardware
### Motors

The robot at Ensta Paris has 40 motors to actuate all 40 joints. It has 7 per arm, 5 per hands, 8 in the head and 8 in the base. Each motor is connected to a board which deals with low-level control and EtherCAT communication. They are controlled at 4Khz by teir EtherCAT boards, i.e the DSPs (for Digital Signal Processing unit). Although all motors in the head and the base are simple **brushed/brushless DC motors**, motors in the arms and hands are **Series Elastic Actuators**, used to enable fine force sensing.

You can read more about series elastic actuators [here](http://www.cc.gatech.edu/fac/Chris.Atkeson/legs/jh1c.pdf), [here](http://groups.csail.mit.edu/lbr/hrg/1995/mattw_ms_thesis.pdf) or [here](http://www.ihmc.us/users/jpratt/publications/2004_Pratt_SPIE.pdf).
### DSP control boards
The DSPs have been designed and programmed by Meka robotics. They contain all necessary configuration for the electronic components, a communication applet (EtherCAT applet) and an internal PID for controlling the motor. Source code can be found in mekabot/m3meka/m3dsp.

> Note: There is a python script called m3qa_batch_firmware_update.py that allow to upgrade the DSP's firmware, but I do not recommend anyone non (highly) advised to use it (can potentially cause hardware issues).

### EtherCAT
This is the communication protocol of the robot, it's a realtime ethernet protocol (i.e. fast, allocation-free and deterministic). Every motor is connected to the realtime computer (meka-mob) via ethernet cables and firewire cables (EtherCAT over firewire). The official EtherCAT by berkhoff : http://www.beckhoff.com/EtherCAT/
On the realtime computer runs the 'EtherCAT master', the core program. We use an open source implementation of the EtherCAT Master program by [IgH](http://etherlab.org/en/ethercat/). When you launch the EtherCAT master (it is launched automatically on the robot), it scans the ethernet bus to find all the EtherCAT slaves connected (i.e. the boards connected to the motors) via a patch ethernet driver, ping all the slaves and preallocate all necessary resources to be real-time safe.
To see the master status (might need sudo privileges):
```bash
ethercat master
```
To see all the connected slaves and their status:
```bash
ethercat slaves # Maybe ethercat slaves -v for more verbose
# Or the shortcut (you need to have m3 installed)
lsec
```
### Arms
![Meka A2 Arm](http://venturebeat.files.wordpress.com/2013/12/meka-arm.jpg)
- Codename :  a2r4
  - right arm : ma17
  - left arm : ma20
- Number of DOF = 7 (redundant robot)
- Motors : Series Elastic Actuators, highly back drivable, last 2 joint (the wrist) is a differential joint.
- Human size - Same joint limits as human arm
- Weight = ~7kg each + hand (0.8kg)
- Inverse kinematics = ikfast (python bindings)
- Inverse dynamics = mass model only (gravity compensation) via KDL (complete inverse dynamics model is unstable due to high noise in the velocities and accelerations)
- Controlled in **torque**, **position** and (new) **velocity** (position commands are converted into a torque command)
- Payload max : <2Kg
- Precision : <1 cm
- Repeatability: good enough for reinforcement learning

### Hand
- Codename :  h2r5
  - right hand : mh16
  - left hand : mh28
- Number of DOF = 5 (2 thumb, and 1 per finger)
- Motors : Series Elastic Actuators, used to enroll the finger tendon. 
- Adapts to any shape due (enrolls around the object)
- Weight = ~ 0.8kg
- Controlled in **torque** and **position** (position commands are converted into a torque command)
- Payload max : <1Kg
- Repeatability: bad due to high friction.

### Head
- Codename :  s2r1
- Number of DOF = 5 (2 thumb, and 1 per finger)
- Motors : Brushless DC motors, no SEAs
- Controlled in **position only** (no series elastic actuators)

### Base + Lift
- Codename :  mb2
- Quasi holonomic behavior
- Number of DOF = 3 (X,Y and \theta)
- Can sense the applied forces (estimation base on current)
- Wheels by Holomni LLC (also bought by Google in december 2013)

### Computers
The Meka robot at Ensta ParisTech has two embedded computers on it : **meka-mob** and **meka-moch**.
#### Meka-mob (RTPC)
This is the real-time PC, i.e an Ubuntu 14.04 LTS (oct 2014) Desktop with an RTAI-patched kernel. It has two Ethernet controllers : a simple one connected to internet via the internal router, and one connected to the EtherCAT boards (via multiple series of switches). This computer has the responsability to launch the real-time server, updating status from all the EC boards, receiving commands and parameters from connected clients (via TCP/IP) and sending them to the EC boards. 

This computer is used to develop real-time components (inside the 1Khz real-time loop) and shared memory components. You might also integrate some ROS real-time components in your M3 components or shared memory components (see http://wiki.ros.org/realtime_tools).

> Note: This computer should **not** be loaded with 'heavy' stuff (computationally speaking), graphics and crazy web browsing while running the real-time server. It might cause unexpected interruptions and break the real-time loop, and hang the computer (the robot will crash).

#### Meka-moch (PC)
This is the 'extra' computer. It's a standard Ubuntu 14.04 LTS (oct 2014) with the m3 python API installed. The eyes cameras and the kinect(s) are connected to this computer.

This computer is used to develop python demos, using the cameras, kinects, microphones or whatever and ROS related stuff that should not be inside the real-time computer.


## Software
### Mekabot M3 software
This schema illustrates the different software components that runs on the robot, from the very end user interface to low-level motor control:
[![M3 Software architecture](https://docs.google.com/drawings/d/1k4oZghxkHyQGyi3N8z-f0xTZeHwmo-l4X53LVXrH0QY/pub?w=1121&amp)](http://goo.gl/OcNcvQ)
### RTAI for real-time applications
RTAI (for Real Time Application Interface) is a real-time library used to write applications with strict timing constrains. It requires to compile a linux kernel with a provided patch (for the Linux PIPE and HAL) to 'enable' the real-time features. Strict timing constraints is a necessary condition for stable control. Without this patch, there would be a lot of delays, especially getting the status/command/param from the EC Board at 4Hkz, making the robot unstable and unusable.

You can read more about RTAI on the [official website](https://www.rtai.org/), or reading the (old) [user manual](http://www.cs.ru.nl/lab/rtai/RTAI_User_Manual-3.4.pdf) and the README files in the repo.

On the Meka we use [ShabbyX's unofficial RTAI](https://github.com/ShabbyX/RTAI), who provides helpful updates and support.

We can checkout the [support mailing list](https://mail.rtai.org/pipermail/rtai/?&MMN_position=21:21) to ask questions.


### The kernel module m3ec.ko
This module is the bridge between kernel and user space. It is used to get all the information (current torque, current 'current' etc) from the slaves (the EtherCAT boards connected to the motors) and writes them to a shared memory (an real-time shared memory). Ideally is would run at the same speed as the m3rt_server but due to the amount  of slaves (~40) on the robot, it requires that the data is split into multiple Ethernet frames, called domains. On the robot, we need to have 4 domains to get all the 40 slaves data, meaning that the kernel module needs to run at 4 times the m3rt_server speed, 4 x 1Khz = **4Khz**. Then every 4 runs, it sends a signal via a **synchronization semaphore** to the m3rt_server (which wait on this semaphore) to notify him that all the data has been updated.

This synchronization semaphore necessitates that RTAI is perfectly configured so that system interruptions does not interfere with it. If RTAI is badly configured, then the m3rt_server might receive the "ok" signal with a lot of delay, breaking the real-time constrain.
You can read a bit more on interruptions in the RTAI world in the source dir of RTAI : $RTAI_SRC/base/arch/x86/calibration/README_SMI.

### The M3 Real-Time Server (or m3rt_server)
The m3rt_server is the robot's realtime system. It creates all the robot's components (motors, joints,  actuators, arms, humanoid etc) and starts the realtime loop that is basically just getting status from every component and sending commands.
#### M3Components
M3Component is the base class of everything that runs on the meka.
It contains a few functions:

* **ReadConfig()**: each component must have a config file (yaml format), this function is called first for all components to get the config (pids, dependant components, names etc)
* **Startup()**: Initialisation. That's usually where you should start threads if you need some. Called once for every components.
* **LinkDependentComponents()**: Get the unique pointer to each components you'd like to get status/send command from/to.
* **StepStatus()**: update status. Should be implemented.
* **StepCommand()**: compute commands. Should be implemented.
*  **Shutdown()**: last thing to be called. That's where you free the memory.


You can check the [m3 cmake package creator](https://github.com/ahoarau/m3-cmake-project-creator) as a tutorial. 

#### TCP/IP server
The TCP/IP server allows the realtime server (m3rt_server) to receive commands (status and param) from remote clients, like **meka-moch**, using the M3 Python API. Protobufs iis the communication protocol used to send data beetween C++ and python, via serialization.

> Note: The speed of the TCP/IP server as been greatly improved to 250Hz (originally it was ~60Hz). Delays are quite short if the computer is connected with ethernet, so you can prototype some control laws, but don't except high speed and robust control with python and network sharing.

> Note2: Even though no remote c++ API has been written, you can still 'connect' remotely to the robot with c++ using the protobuf. But you'll have to write your own API.

#### Google Protobufs

[Official Google Protobuf wiki](https://developers.google.com/protocol-buffers/docs/overview)

Protobuf is nice mechanism to serialize data. It uses a .proto file (language agnostic) where you can define messages you'd like to share (floats, strings etc), and generates c++, python (more are available) to encode/decode data.

All status, parameters and commands are defined as a protobuf message. When you subscribe to the status of a component (ex: M3Humanoid, the highest level), you actually tell the m3rt_server to send you the **serialized** version of the protobuf from the c++ (realtime), that it automatically **deserializes** to put in the generated protobuf python, on the client side (i.e your side). It's the same for param and command.

The same way, when you send command to the robot, you actually change the attributes of the generated protobuf python class ( named *_pb2.py ), then when you call proxy.step(), it serializes this class (it's made for that), send it via standard TCP/IP, the m3rt_server (c++) receives the serializes data, deserializes it to put on it's own c++ protobuf class (generated from the .proto).




