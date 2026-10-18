FROM ros:jazzy-ros-base

# CI（.github/workflows/ci.yml）と同じツール一式 + 開発用の最低限
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    sudo \
    bash-completion \
    python3-colcon-common-extensions \
    python3-rosdep \
    python3-vcstool \
    ros-jazzy-ament-cmake-gtest \
    ros-jazzy-ament-lint-auto \
    ros-jazzy-ament-lint-common \
    ros-jazzy-rviz2 \
    ros-jazzy-xacro \
    ros-jazzy-robot-state-publisher \
    ros-jazzy-joint-state-publisher-gui \
    && rm -rf /var/lib/apt/lists/*

# ros:jazzy イメージ既定の非 root ユーザー（ubuntu, uid 1000）に sudo を許可
RUN echo "ubuntu ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/ubuntu \
    && chmod 0440 /etc/sudoers.d/ubuntu

USER ubuntu
# Qt が要求する 0700 のランタイムディレクトリ（WSLg の runtime-dir は 0777 なので別に用意）
RUN mkdir -m 0700 -p /tmp/runtime-ubuntu
RUN rosdep update \
    && echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc \
    && echo "[ -f ~/rabbits_ws/install/setup.bash ] && source ~/rabbits_ws/install/setup.bash" >> ~/.bashrc \
    && echo "source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash" >> ~/.bashrc

WORKDIR /home/ubuntu/rabbits_ws
