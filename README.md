# ROS Tutorials

Walking through tutorial at http://wiki.ros.org/ROS/Tutorials

### Requires 
`sudo apt-get install ros-indigo-ros-tutorials`
`sudo apt-get install ros-indigo-rqt ros-indigo-rqt-common-plugins`

### Basic Info
* Nodes are executables ROS uses to communicate with other nodes by subscriping to topics
  * `rosnode` to interact with nodes (get info, list, etc)
* Messages are ROS data types used when subscribing or publishing to a topic
* Topics are an ROS element that nodes can publish or subscribe to to receive messages
* Master is the name of the main ROS service which lets nodes find each other
  * `roscore` is command to run master + rosout + parameter server
  * `rosout` is a stdout/stderr equivalent
  * `rosrun` to run nodes within packages

### Editor
`rosed [package_name] [file_name]`
Setup with `export EDITOR='emacs -nw'`
`roscp [package_name] [file_to_copy_path] [copy_path]`

### Creating a workspace
```
source /opt/ros/indigo/setup.bash
mkdir -p ~/ros_tutorials/src
cd ~/ros_tutorials/src
catkin_init_workspace
cd ..
catkin_make
source devel/setup.bash
```

### Creating a package
```
cd ~/ros_tutorials/src
catkin_create_pkg beginner_tutorials std_msgs rospy roscpp
cd beginner_tutorials
```
Update info in `package.xml`

### Building Packages
```
cd ~/ros_tutorials
catkin_make
```

### Running Nodes
Must run `roscore` first (in different terminal)
Running with custom naming:  
`rosrun turtlesim turtlesim_node __name:=my_turtle`
To visualize node diagram:  
`rosrun rqt-graph rqt-graph`
Use `rostopic` to get into on ROS topics:  
`rostopic list`  
`rostopic echo /turtle1/cmd_vel`  
`rostopic type /turtle1/cmd_vel` to get name of message type
Use `rosmsg` for more information on message
`rosmsg show geometry_msgs/Twist`
To publish data on to a topic:
`rostopic pub -1 /turtle1/cmd_vel geometry_msgs/Twist -- '[2.0, 0.0, 0.0]' '[0.0, 0.0, 1.8]'`
To graph data over time:
`rosrun rqt_plot rqt_plot`

### Services
Services allow nodes to send a request and get a response
Use `rosservice` to list, call, type, find, and uri
`rosservice type /clear`
`rosservice call /clear`
`rosservice type /spawn| rossrv show`
`rosservice call /spawn 2 2 0.2 ""`

### Parameter Server
Use `rosparam` to store and update data on the ROS Parameter Server
`rosparam` can set, get, load, dump, delete, and list
`rosparam list`
`rosparam set /background_g 200`
`rosparam dump params.yaml`
`rosparam load params.yaml copy`
`rosparam get /copy/background_g`

### Debugging with rqt_console
`rqt_console` attaches to ROS's logging framework to display output from nodes.
`rosrun rqt_console rqt_console`
`rqt_logger_level` is used to change verbosity level
`rosrun rqt_logger_level rqt_logger_level`

### Using roslaunch
`roslaunch` starts nodes as defined in a launch file. This example will launch two turtlesim and mimic actions from turtlesim1 to turtlesim2
`roscd beginner_tutorials`
`mkdir launch && cd lauch`
Take a look at `turtlemimic.launch` file
`roslaunch beginner_tutorials turtlemimic.launch`
`rostopic pub /turtlesim1/turtle1/cmd_vel geometry_msgs/Twist -r 1 -- '[2.0, 0.0, 0.0]' '[0.0, 0.0, -1.8]'`

### Basics of msg and srv
`msg` files define the fields of ROS messages and are used to generate source code for messages
`srv` files define a service's request and response

#### Using msg 
Example of a msg that uses a Header, a string primitive, and two other msgs:
```
Header header
  string child_frame_id
  geometry_msgs/PoseWithCovariance pose
  geometry_msgs/TwistWithCovariance twist
```

Look at beginner_tutorials/msg for a basic msg example.
Remeber to update `package.xml` and enable the `message_generation` and `message_runtime` dependencies.
Also need to add `message_generation` to the required components in `CMakeLists.txt` and `message_runtime` to the `CATKIN_DEPENDS` section.
Add message file name to the `add_message_files` section of `CMakeLists.txt`
Make sure `generate_messages` is called in `CMakeLists.txt`
Verify message with `rosmsg show beginner_tutorials/Num`

#### Using srv
srv files separate the request and reponse with a `---` line:
```
int64 A
int64 B
---
int64 Sum
```
Take a look at beginner_tutorials/srv for a basic srv example.
Remeber to update `package.xml` and enable the `message_generation` and `message_runtime` dependencies.
Also need to add `message_generation` to the required components in `CMakeLists.txt` and `message_runtime` to the `CATKIN_DEPENDS` section.
Add service file name to the `add_service_files` section of `CMakeLists.txt`
Make sure `generate_messages` is called in `CMakeLists.txt`
Verify service with `rossrv show beginner_tutorials/AddTwoInts`

### Publisher Node
The example node at `beginner_tutorials/src/talker.cpp` will continually broadcast a message.
Node activities:
* Init ROS system
* Advertise publishing std_msgs/String messages on the `chatter` topic to the master
* Loop while publishing messages to `chatter` 10 times a second

### Subscriber Node
The example node at `beginner_tutorials/src/listener.cpp` will subscribe to and receive messages from the talker node.
Node activities:
* Initialize the ROS system
* Subscribe to the `chatter` topic
* Spin, waiting for messages to arrive
* When a message arrives, the `chatterCallback()` function is called

### Building Nodes
Based on the above publisher and sucscriber example, add the following to `CMakeLists.txt`
```
include_directories(include ${catkin_INCLUDE_DIRS})

add_executable(talker src/talker.cpp)
target_link_libraries(talker ${catkin_LIBRARIES})
add_dependencies(talker beginner_tutorials_generate_messages_cpp)

add_executable(listener src/listener.cpp)
target_link_libraries(listener ${catkin_LIBRARIES})
add_dependencies(listener beginner_tutorials_generate_messages_cpp)
```
In the catkin workspace run `catkin_make`, may need `--force-cmake` option when adding a new pkg

### Running the Publisher and Subscriber
Make sure `roscore` is running and workspace is sourced.
`rosrun beginner_tutorials talker`
`rosrun begineer_tutorials listener`
Investigate with `rostopic list` and `rqt_graph`

### Using a Service and Client Nodes
#### Service
The example at `beginner_tutorials/src/add_two_ints_server.cpp` creates a node which will receive two ints and return the sum
#### Client
The example at `beginner_tutorials/src/add_two_ints_client.cp` creates a node which will send two ints to the server and receive the sum response.
#### Building
Add the following to `CMakeLists.txt`
```
add_executable(add_two_ints_server src/add_two_ints_server.cpp)
target_link_libraries(add_two_ints_server ${catkin_LIBRARIES})
add_dependencies(add_two_ints_server beginner_tutorials_gencpp)

add_executable(add_two_ints_client src/add_two_ints_client.cpp)
target_link_libraries(add_two_ints_client ${catkin_LIBRARIES})
add_dependencies(add_two_ints_client beginner_tutorials_gencpp)
```
In the catkin workspace run `catkin_make`, may need `--force-cmake` option when adding a new pkg
#### Running the Client Server Example
Make sure `roscore` is running.
`rosrun beginner_tutorials add_two_ints_server`
`rosrun beginner_tutorials add_two_ints_client 1 3`

### Record and Play Back Data
Data from topics can be recorded into a bag file using `rosbag`.
Record all topics: `rosbag record -a`
`rosbag info <bagfile>`
Playback data: `rosbag play <bagfile>`
Record specific data: `rosbag record -O velandpose /turtle1/cmd_vel /turtle1/pose`

### Troubleshooting with roswtf
`roswtf` examines the system and tries to find problems.


