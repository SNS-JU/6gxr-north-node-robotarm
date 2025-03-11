
# Based image as ROS2 Jazzy for Ubuntu Jammy(22.xx)
FROM osrf/ros:jazzy-desktop

#package installation
RUN apt-get update && apt-get install -y \
    ros-jazzy-ur-robot-driver \
    iputils-ping nano net-tools \
    python3-rosdep python3-colcon-common-extensions

#Workspace setup
#WORKDIR /root/ros2_ur_driver_ws
#COPY . .

#RUN rm -f /etc/ros/rosdep/sources.list.d/20-default.list && \
#    rosdep init && \
#    rosdep update

#build
#RUN colcon build --symlink-install

# ROS2 settings
#RUN echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc

CMD ["bash"]
