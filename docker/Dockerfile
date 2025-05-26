FROM palrobotics/public-tutorials-alum-devel

# Update sources to use Ubuntu 22.04 repositories
RUN echo "deb http://archive.ubuntu.com/ubuntu/ jammy main restricted universe multiverse" > /etc/apt/sources.list && \
    echo "deb http://archive.ubuntu.com/ubuntu/ jammy-updates main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb http://archive.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb http://archive.ubuntu.com/ubuntu/ jammy-backports main restricted universe multiverse" >> /etc/apt/sources.list

ENV DEBIAN_FRONTEND=noninteractive

# Install core dependencies
RUN apt-get update && \
    apt-get install -y \
        curl \
        gnupg \
        lsb-release \
        build-essential \
        libncurses-dev \
        git \
        vim \
        zsh \
        wget

# Install Gazebo
RUN curl -sSL http://get.gazebosim.org | sh

# Add ROS 2 GPG key and repository
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /tmp/ros.key && \
    gpg --dearmor /tmp/ros.key && \
    mv /tmp/ros.key.gpg /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" > /etc/apt/sources.list.d/ros2.list && \
    rm /tmp/ros.key

# Install ROS 2 package
RUN apt-get update && \
    apt-get install -y joint-state-publisher-gui && \
    apt-get install -y liburdfdom-tools && \
    apt-get install -y ros-humble-gazebo-ros && \
    apt-get install -y ros-humble-controller-manager && \
    apt-get install -y ros-humble-moveit-configs-utils && \
    apt-get install -y ros-humble-moveit-ros-move-group && \
    apt-get install -y ros-humble-play-motion2 && \
    apt-get install -y ros-humble-point-cloud-transport && \
    apt-get install -y ros-humble-realsense2-* && \
    apt-get install -y ros-humble-topic-tools && \
    apt-get install -y python3-tqdm && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create new user "pal" with home directory
RUN useradd -m -s /bin/bash pal && \
    echo "pal ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/pal && \
    chmod 0440 /etc/sudoers.d/pal

# Download the Oh My Zsh install script to /tmp/install.sh
RUN curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -o /tmp/install.sh && \
    chmod +x /tmp/install.sh

# Install Oh My Zsh for the pal user
RUN su - pal -c "ZSH=/home/pal/.oh-my-zsh /tmp/install.sh --unattended" && \
    git clone https://github.com/zsh-users/zsh-autosuggestions /home/pal/.oh-my-zsh/custom/plugins/zsh-autosuggestions && \
    git clone https://github.com/zsh-users/zsh-syntax-highlighting /home/pal/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting && \
    sed -i 's/plugins=(git)/plugins=(git zsh-autosuggestions zsh-syntax-highlighting)/g' /home/pal/.zshrc && \
    chown -R pal:pal /home/pal/.oh-my-zsh /home/pal/.zshrc

# Set default shell to zsh for pal
RUN chsh -s /bin/zsh pal

RUN echo "export TERM=xterm-256color" >> /home/pal/.zshrc && \
    echo 'PROMPT="%F{208}gz%f ${PROMPT}"' >> /home/pal/.zshrc && \
    echo "source /opt/ros/humble/setup.zsh" >> /home/pal/.zshrc && \
    echo "source /opt/pal/alum/setup.zsh" >> /home/pal/.zshrc && \
    echo "source /home/pal/ros2_ws/install/setup.zsh" >> /home/pal/.zshrc && \
    echo "" >> /home/pal/.zshrc && \
    echo "# argcomplete for ros2 & colcon" >> /home/pal/.zshrc && \
    echo 'eval "$(register-python-argcomplete3 ros2)"' >> /home/pal/.zshrc && \
    echo 'eval "$(register-python-argcomplete3 colcon)"' >> /home/pal/.zshrc && \
    chown pal:pal /home/pal/.zshrc

# Create workspace directory with permissions
RUN mkdir -p /home/pal/ros2_ws/src && \
    chown -R pal:pal /home/pal/ros2_ws && \
    chmod -R 755 /home/pal/ros2_ws

# Copy and import repositories
COPY ./pro.repos /tmp/pro.repos

RUN pip3 install git+https://github.com/nzlzcat/vcstool

RUN vcs import /home/pal/ros2_ws/src < /tmp/pro.repos && \
    chown -R pal:pal /home/pal/ros2_ws/src

RUN rosdep init

USER pal

# rosdep
RUN rosdep update && \
rosdep install --from-paths /home/pal/ros2_ws/src --ignore-src -r -y

WORKDIR /home/pal/ros2_ws
# compile unsetting environment variable, due to missing not-public packages
SHELL ["/bin/zsh", "-c"]
RUN source /opt/ros/humble/setup.zsh && \
    source /opt/pal/alum/setup.zsh && \
    unset PAL_DISTRO && \
    colcon build

WORKDIR /home/pal

# Override gosu
ENTRYPOINT []

CMD ["zsh", "-l"]