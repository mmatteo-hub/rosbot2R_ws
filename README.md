# ROSbot workspace
This folder contains the files for starting the sensors of the [ROSbot 2R](https://husarion.com/manuals/rosbot/), from [Husarion](https://husarion.com/). <br>
This [README](/README.md) contains the instructions to setup the ROSbot and to make it usable when launching ROS among multiple machine. It is also shown how to broadcast the time over the Wi-Fi in order to have all the agents synchronised with the master. <br><br>

The [folder](/src/) of the workspace has to be put in the `ros_ws` of each robot needed and properly built through `catkin_make`.

## ROSbot start packaged
The starting packages can be download from the official [Github Page](https://github.com/husarion/). <br>
In particular the packages needed are:
* [rosbot_ekf](https://github.com/husarion/rosbot_ekf)
* [rosbot_description](https://github.com/husarion/rosbot_description)

### IMPORTANT
All the files are setup with a namespace (`robot_0` in this case), becuase there was the need to use multiple robots of the same type. Remeber to change the files parameter and the folder names, in particular:
* [start_pkg](/src/start_pkg/):
    * [start.launch](/src/start_pkg/launch/start.launch): `robot_ns` line 3
* [rosbot_navigation](/src/rosbot_navigation/):
    * [costmap_common_params.yaml](/src/rosbot_navigation/config/costmap_common_params.yaml): change the namespace of the following parameters:
        * `map_topic`: line 4
        * `laser_scan_sensor`: `sensor_frame` and `topic` line 7
        * `robot_base_frame`: line 9
        * `map_topic`: line 13
    * [exploration.yaml](/src/rosbot_navigation/config/exploration.yaml): change the namespace of the following parameters:
        * `robot_base_frame`: line 7
        * `map_topic`: line 14

# ROSbot setup
These instructions are needed to setup the ROSbot in order to make the robots work correctly.

## .bashrc file
Before starting check the `.bashrc` file and change the following settings. <br>
Comment the one needed for the `localhost` and set the one for the shared master by adding new lines. <br>
At the end you should have something as:
```bash
# Local host master
#export ROS_MASTER_URI=http://localhost:11311
#export ROS_HOSTNAME=localhost

# Shared master with rosbot
export ROS_MASTER_URI=http://PC_IP:11311
export ROS_IP=PC_IP
```
where the `PC_IP` can be retrieved with `hostname -I` once connected to the Wi-Fi. <br>
Remember to source the `.basrch` file once modified otherwise the changes will not be taken into account.<br><br>

Once in this situation, when launching the `roscore` on the master PC you can see some topics also from the other machines which are sharing the master.

## Broadcast the time on the Wi-Fi
For the ROSbot in the Lab sometimes the clock is lost so the program will have problems in running cause of mismatch in the TF with the timestamps. <br>
To avoid this, the master PC can broadcast its own clock over the Wi-Fi in order to have all the devices sharing the same. To do so follow these steps:
* on the master PC create a dedicated file `ntp.conf` in the `/etc` folder and set it as:
```bash
# Use the local clock
server 127.127.1.0 prefer
fudge  127.127.1.0
driftfile /var/lib/ntp/drift
broadcastdelay 0.008

# Give localhost full access rights
restrict default

# Give machines on our network access to query us
restrict PC_IP.0 mask 255.255.255.0 nomodify notrap

broadcast PC_IP.0
```
where `PC_IP` is `nnn.nnn.nnn.0`: important to set to `0` the last one set.

* on each of the agent that has to retrieve the clock create a file `ntp.conf` int the `/etc` folder and write this inside
```bash
# Point to our network's master time server
server PC_IP iburst

restrict default

driftfile /var/lib/ntp/drift

minpoll 4
maxpoll 5
```
where `PC_IP` is the complete IP of the master PC over the wifi (retrieved with `hostname -I` form the master PC).

Once all the files are set up, reboot the agents (`sudo reboot now`) to be sure teh changes are applied and the clock will be correctly shared.