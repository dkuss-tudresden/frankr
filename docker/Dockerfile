
FROM ros:kinetic

LABEL maintainer="Nicolai Anton Lynnerup nily@dti.dk & Kim Lykke Nørregaard kimn@dti.dk & Lærke Fabricius llfa@dti.dk"

SHELL ["/bin/bash", "-c"]

# Setup .bashrc
RUN echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> ~/.bashrc \
	&& /bin/bash -c "source ~/.bashrc"

################
# Dependencies #
################

RUN apt-get update && apt-get install -y --no-install-recommends \
		iproute2 \
		iputils-ping \
		mlocate \
		nano \
		gedit \ 
		net-tools \
        python3-dev \
	    python3-pip \
	    python3-yaml \
        python3-empy \
		python3-setuptools \
        wget \
		build-essential \
		libpoco-dev \
		libeigen3-dev \
		&& updatedb \
		&& apt-get remove "*libfranka*"

RUN pip3 install "pybind11[global]"
RUN pip3 install wheel

################
# CMake 3.18.4 #
################

RUN apt-get update \
	&& apt-get install -y apt-transport-https ca-certificates gnupg software-properties-common \
	&& wget --no-check-certificate -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | gpg --dearmor - | tee /etc/apt/trusted.gpg.d/kitware.gpg >/dev/null \
	&& apt-add-repository 'deb https://apt.kitware.com/ubuntu/ xenial main' \
	&& apt-get update \
	&& apt-get install kitware-archive-keyring \
	&& rm /etc/apt/trusted.gpg.d/kitware.gpg \
	&& apt-get install -y cmake

################
# ROS Packages #
################

RUN apt-get update && apt-get install -y --no-install-recommends \
		ros-${ROS_DISTRO}-moveit \
		ros-${ROS_DISTRO}-joint-state-controller \
		ros-${ROS_DISTRO}-joint-trajectory-controller \
		ros-${ROS_DISTRO}-effort-controllers \
		ros-${ROS_DISTRO}-position-controllers

RUN pip3 install rospkg catkin_pkg

###############
# Eigen 3.3.7 #
###############

RUN cd /tmp/ \
    && wget -q --show-progress --progress=bar:force https://gitlab.com/libeigen/eigen/-/archive/3.3.7/eigen-3.3.7.tar.gz \
	&& tar -xzf eigen-3.3.7.tar.gz \
	&& cd eigen-3.3.7 \
	&& mkdir build && cd build \
	&& cmake .. && make install \
    && cd /tmp/ && rm eigen-3.3.7.tar.gz

#############
# Workspace #
#############

RUN mkdir -p /home/workspace/
WORKDIR /home/workspace

# Create Catkin WS
RUN mkdir -p catkin_ws/src/

# Set environment variables
ENV PYTHONPATH=$PYTHONPATH:/home/workspace/catkin_ws/devel/lib

#############
# libfranka #
#############

RUN git clone --recursive --branch 0.8.0 https://github.com/frankaemika/libfranka.git \
	&& cd libfranka \
	&& git submodule update \
	&& mkdir build && cd build \
	&& cmake -DCMAKE_BUILD_TYPE=Release .. \
	&& cmake --build .

##############
# Franka ROS #
##############

RUN cd catkin_ws \
	&& /bin/bash -c "source ~/.bashrc" \
	&& /bin/bash -c ". /opt/ros/${ROS_DISTRO}/setup.sh; catkin_init_workspace src" \
	&& git clone --recursive https://github.com/frankaemika/franka_ros src/franka_ros \
	&& apt-get update \
	&& rosdep install --from-paths src --ignore-src --rosdistro ${ROS_DISTRO} -y \
	&& /bin/bash -c ". /opt/ros/${ROS_DISTRO}/setup.bash; catkin_make -DPYTHON_EXECUTABLE=/usr/bin/python3 -DCMAKE_BUILD_TYPE=Release -DFranka_DIR:PATH=/home/workspace/libfranka/build" \
	&& /bin/bash -c "source ~/.bashrc"

##########
# MoveIt #
##########

RUN wget -q --show-progress --progress=bar:force https://raw.githubusercontent.com/dti-research/panda_moveit_config/kinetic-devel/config/panda_arm.xacro \
	&& cp panda_arm.xacro /opt/ros/${ROS_DISTRO}/share/panda_moveit_config/config/

##########
# Frankr #
##########

RUN cd catkin_ws/src/ \
    && git clone https://github.com/dti-research/frankr.git \
    && cd .. \
    && /bin/bash -c ". /opt/ros/${ROS_DISTRO}/setup.bash; catkin_make" \
	&& echo "source /home/workspace/catkin_ws/devel/setup.bash" >> ~/.bashrc \
	&& /bin/bash -c "source ~/.bashrc"