# Line Follower PA

## Explanation of Code

### Video of Code Execution

See [my robot following a yellow line](https://drive.google.com/file/d/1WGkbqJCFMbHdIANs4VHX3sLer-FZ7b6b/view?usp=share_link).

### How the Code Runs

When the command `roslaunch line_follower_pa linemission.launch model:=waffle` is executed on the command line in a catkin workspace, the `linemission.launch` file uses the `gazebo_ros` package and the `lfm1.world` file in the `worlds` directory to begin a gazebo simulation with the waffle robot in a world with a yellow line.

Next, if `rosrun line_follower_pa line_follower.py` is executed, also on the command line and in a catkin workspace, the `follower` node is connected via `roscore` to the simulated robot, which then starts following the yellow line.

### Explanation of line_follower.py

The logic of the line-following algorithm itself is in `line_follower.py`. There, a `Follower` class is defined and initialized, which subscribes to the `/camera/rgb/image_raw` topic, processes the `Image` messages it receives therefrom, and publishes appropriate `Twist` messages to the `/cmd_vel` topic, causing the robot to follow the yellow line.

The image processing and the publishing of `Twist` messages occurs in the `image_callback` function, which is executed each time the `follower` node receives an `Image` message from the `/camera/rgb/image_raw` topic.

The `image_callback` function, then, first converts the image into a format that is compatible with the `cv2` package, after which it changes the image format from RGB to HSV. This is because: (i) we want to use the `cv2` package to filter out the yellow color of the line we want to follow, by creating a binary image that basically has a 1 for where the line is and a 0 for all other cells; and (ii) we cannot use a RGB image to filter out yellow reliably in this way, because RGB might mistakenly construe as yellow what is not and vice versa because its values can be corrupted by the brightness of objects (cf. PRR p.201). So to compensate for this fault of RGB, we use the HSV format instead, which separates the brightness (or 'Value', which is the 'V' in 'HSV') from the color itself (the 'Hue').

This is all accomplished in the parts of the code under the comments `# get image from camera` and `# filter out everything that's not yellow`.

Next, the code isolates a band of yellow pixels for the robot to follow, so that it does not look too far ahead to a portion of the line that is too far from it (cf. PRR p.204). This portion of the code is under the comment `# clear all but a 20 pixel band of the image`.

Finally, the rest of the `image_callback` function identifies a centroid, or a central point in the blob that is the yellow line, that we instruct the robot to follow by publishing appropriate linear and angular velocities with a `Twist` message.

After the `image_callback` and the definition of the `Follower` class, the code simply initializes: (i) a ROS node, calling it `follower`, and (ii) an instance of the `Follower` class, which causes the proper subscription and publication to be set up via the constructor `__init__` and the `image_callback` function to be executed whenever an `Image` message is received by the `follower` node. Finally, `rospy.spin()` just causes the node to run until a SIGINT is sent via a control-C command.

