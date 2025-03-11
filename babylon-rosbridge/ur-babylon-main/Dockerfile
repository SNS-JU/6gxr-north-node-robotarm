# Use ROS2 Rolling as the base image
FROM osrf/ros:rolling-desktop
# Install necessary packages
RUN apt-get update
RUN apt-get install -y \
    ros-rolling-ros2-control \
    ros-rolling-ros2-controllers \
    ros-rolling-rosbridge-server
# Create the ROS user with sudo permissions
RUN useradd -ms /bin/bash rosuser && echo "rosuser ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
# Adjust permissions for the log directory
RUN mkdir -p /home/rosuser/.ros/log && chown -R rosuser:rosuser /home/rosuser/.ros
# Switch to rosuser
USER rosuser
