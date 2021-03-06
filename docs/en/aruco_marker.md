# ArUco marker detection

> **Info** Marker detection requires the camera module to be correctly plugged in and [configured](camera.md).

`aruco_detect` module detects ArUco markers and publishes their positions in ROS topics and as [TF frames](frames.md).

This is useful in conjunction with other positioning systems, such as [GPS](gps.md), [Optical Flow](optical_flow.md), PX4Flow, visual odometry, ultrasonic ([Marvelmind](https://marvelmind.com)) or UWB-based ([Pozyx](https://www.pozyx.io)) localization.

Using this module along with [map-based navigation](aruco_map.md) is also possible.

## Setup

Set the `aruco` argument in `~/catkin_ws/src/clever/clever/launch/clever.launch` to `true`:

```xml
<arg name="aruco" default="true"/>
```

For enabling detection set the `aruco_detect` argument in `~/catkin_ws/src/clever/clever/launch/aruco.launch` to `true`:

```xml
<arg name="aruco_detect" default="true"/>
```

For the module to work correctly the following parameters should be set:

```xml
<param name="length" value="0.32"/>          <!-- length of a single marker, in meters (excluding the white border) -->
<param name="estimate_poses" value="true"/>  <!-- position estimation for single markers -->
<param name="send_tf" value="true"/>         <!-- TF frame creation for markers -->
<param name="known_tilt" value="map"/>       <!-- Marker tilt, explained below -->
```

`known_tilt` should be set to:

* `map` if *all* markers are on the ground;
* `map_flipped` if *all* markers are on the ceiling;
* an empty string otherwise.

You may specify length for each marker individually by using the `length_override` parameter:

```xml
<param name="length_override/3" value="0.1"/>    <!-- marker with id=3 has a side of 0.1m -->
<param name="length_override/17" value="0.25"/>  <!-- marker with id=17 has a side of 0.25m -->
```

## Coordinate system

Each marker has its own coordinate systems. It is aligned as follows:

* the **<font color=red>x</font>** axis points to the right side of the marker;
* the **<font color=green>y</font>** axis points to the top side of the marker;
* the **<font color=blue>z</font>** axis points outwards from the plane of the marker

<img src="../assets/aruco-axis.png" width="300">

## Working with detected markers

Navigation within the marker-based TF frames is possible with `simple_offboard` node.

Sample code to fly to a point 1 metre above marker with id 5:

```python
navigate(frame_id='aruco_5', x=0, y=0, z=1)
```

Sample code to fly to a point 1 metre to the left and 2 metres above marker with id 7:

```python
navigate(frame_id='aruco_7', x=-1, y=0, z=2)
```

Sample code to rotate counterclockwise while hovering 1.5 metres above marker id 10:

```python
navigate(frame_id='aruco_10', x=0, y=0, z=1.5, yaw_rate=0.5)
```

Note that if the required marker isn't detected for 0.5 seconds after the `navigate` command, the command will be ignored.

These frames may also be used in other services that accept TF frames (like `get_telemetry`). The following code will get the drone's position relative to the marker with id 3:

```python
telem = get_telemetry(frame_id='aruco_3')
```

Note that if the required marker isn't detected for 0.5 seconds, the `telem.x`, `telem.y`, `telem.z`, `telem.yaw` fields will contain `NaN`.

## Handling marker detection in Python

The following snippet shows how to read the `aruco_detect/markers` topic in Python:

```python
import rospy
from aruco_pose.msg import MarkerArray
rospy.init_node('my_node')

# ...

def markers_callback(msg):
    print 'Detected markers:':
    for marker in msg.markers:
        print 'Marker: %s' % marker

# Create a Subscription object. Each time a message is posted in aruco_detect/markers, the markers_callback function is called with this message as its argument.
rospy.Subscriber('aruco_detect/markers', MarkerArray, markers_callback)

# ...

rospy.spin()
```

Each message contains the marker ID, its corner points on the image and its position relative to the camera.

---

Suggested reading: [map-based navigation](aruco_map.md)
