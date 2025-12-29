# ==============================================================================
# Multi-Stage Dockerfile for NVIDIA Jetson Orin Cross-Compilation Environment
# ==============================================================================
# Creates a complete x86_64 development environment for cross-compiling ROS2 
# Humble applications targeting NVIDIA Jetson Orin (ARM64). Includes CUDA 12.2,
# JetPack r36.3.0 libraries, and full ROS2 Humble desktop installation.
# ==============================================================================

ARG ARM_PLATFORM=linux/arm64

# ==============================================================================
# Stage 1: Extract ROS2 Humble ARM64 Binaries
# ==============================================================================
# Extracts pre-built ROS2 Humble binaries and dependencies for ARM64 to use
# in the cross-compilation sysroot.
# ==============================================================================
FROM --platform=${ARM_PLATFORM} arm64v8/ros:humble-ros-base AS ros_arm_source

# ==============================================================================
# Stage 2: Extract NVIDIA JetPack Libraries
# ==============================================================================
# Obtains NVIDIA proprietary libraries (CUDA, multimedia APIs, hardware
# acceleration) required for Jetson Orin development.
# ==============================================================================
FROM --platform=${ARM_PLATFORM} nvcr.io/nvidia/l4t-jetpack:r36.3.0 AS jetpack_arm_source

# Add NVIDIA L4T (Linux for Tegra) repositories for JetPack r36.3
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates wget && \
    echo "deb https://repo.download.nvidia.com/jetson/common r36.3 main" > /etc/apt/sources.list.d/nvidia-l4t-apt-source.list && \
    echo "deb https://repo.download.nvidia.com/jetson/t234 r36.3 main" >> /etc/apt/sources.list.d/nvidia-l4t-apt-source.list && \
    wget -qO- https://repo.download.nvidia.com/jetson/jetson-ota-public.asc | apt-key add -

# Download and extract NVIDIA packages without installing to custom locations
# Key packages: multimedia (video encode/decode), core (drivers), DRM (GPU access)
RUN apt-get update && \
    mkdir -p /tmp/extracted && \
    cd /tmp && \
    apt-get download nvidia-l4t-multimedia nvidia-l4t-core nvidia-l4t-multimedia-utils libdrm-dev libdrm2 vpi3-dev libnvvpi3 && \
    for deb in *.deb; do dpkg-deb -x "$deb" /tmp/extracted; done

# Copy extracted libraries and headers to standard paths for sysroot preparation
RUN cp -r /tmp/extracted/usr/lib/aarch64-linux-gnu/* /usr/lib/aarch64-linux-gnu/ || true && \
    cp -r /tmp/extracted/usr/include/* /usr/include/ || true && \
    mkdir -p /opt/nvidia/vpi3 && \
    if [ -d "/tmp/extracted/opt/nvidia/vpi3" ]; then cp -r /tmp/extracted/opt/nvidia/vpi3/* /opt/nvidia/vpi3/; fi

# ==============================================================================
# Stage 3: Obtain x86_64 CUDA 12.2 Host Tools
# ==============================================================================
# Provides CUDA compiler (nvcc) and build tools that run on x86_64 host for
# compiling ARM64 target code.
# ==============================================================================
FROM nvidia/cuda:12.2.0-devel-ubuntu22.04 AS cuda_x86_source

# ==============================================================================
# Final Stage: Ubuntu 22.04 Cross-Compilation Environment
# ==============================================================================
# Assembles the complete development environment with tools, libraries, and
# configurations for Jetson Orin cross-compilation.
# ==============================================================================
FROM ubuntu:22.04

# Sysroot path for ARM64 target libraries
ARG SYSROOT=/opt/nvidia/orin_sysroot

# ------------------------------------------------------------------------------
# System Foundation
# ------------------------------------------------------------------------------

# Install prerequisites for repository management
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    apt-transport-https \
    software-properties-common \
    gnupg \
    && rm -rf /var/lib/apt/lists/*

# Configure timezone non-interactively
ENV TZ=Asia/Shanghai
RUN echo "${TZ}" >/etc/timezone && ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime

# Optimize apt sources for faster downloads
COPY auto_detect_sources.py /tmp/
RUN chmod +x /tmp/auto_detect_sources.py && \
    python3 /tmp/auto_detect_sources.py

# ------------------------------------------------------------------------------
# Development Toolchain
# ------------------------------------------------------------------------------

# Install comprehensive development tools:
# - Version control: git, git-lfs
# - Build tools: gcc, g++, cmake, cross-compilers (aarch64)
# - Debugging: gdb, gdb-multiarch, lldb, strace
# - Network utilities: ping, telnet, tcpdump, iperf
# - Development libraries: OpenSSL, cURL, TBB, JPEG, GLEW, etc.
RUN add-apt-repository -y ppa:git-core/ppa && \
    apt-get update -y && DEBIAN_FRONTEND=noninteractive \
    apt-get install -y \
    --no-install-recommends \
    curl \
    jq \
    git \
    zsh \
    clang-format \
    iputils-ping \
    pkg-config \
    ptpd \
    python3-pip \
    ssh \
    sshpass \
    strace \
    sudo \
    unzip \
    cpio \
    rpm2cpio \
    cabextract \
    wget \
    zip \
    pigz \
    zlib1g-dev \
    gdb \
    vim \
    libcurl4-openssl-dev \
    libssl-dev \
    lcov \
    cppcheck \
    default-jdk \
    doxygen \
    doxygen-doc \
    doxygen-gui \
    libgomp1 \
    libfuse2 \
    tzdata \
    binwalk \
    rsync \
    locales \
    libglew-dev \
    libtbb-dev \
    libjpeg8-dev \
    graphviz \
    highlight \
    gdb-multiarch \
    lldb \
    dnsutils \
    telnet \
    traceroute \
    net-tools \
    iproute2 \
    tcpdump \
    iperf \
    libdwarf-dev \
    tigervnc-tools \
    uuid-runtime \
    mosquitto \
    clang-format-15 \
    git-lfs \
    libccd-dev \
    libglpk-dev \
    libscrypt-dev \
    libcgal-dev \
    libfcl-dev \
    nlohmann-json3-dev \
    liburdfdom-dev \
    libzmq3-dev \
    libgflags-dev \
    libglfw3-dev \
    libacl1-dev \
    gcc-aarch64-linux-gnu \
    g++-aarch64-linux-gnu \
    fuse && apt-get -y clean

# Install GCC 13 for C++23 support and improved optimizations
RUN add-apt-repository -y ppa:ubuntu-toolchain-r/test && \
    apt-get update -y && \
    apt-get install -y gcc-13 g++-13 && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/*

# Configure GCC alternatives (GCC 11 and 13) with priority settings
# Higher priority (130) makes GCC 13 the default
RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 110 \
    --slave /usr/bin/g++ g++ /usr/bin/g++-11 && \
    update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-13 130 \
    --slave /usr/bin/g++ g++ /usr/bin/g++-13 && \
    update-alternatives --install /usr/bin/gcov gcov /usr/bin/gcov-11 110 && \
    update-alternatives --install /usr/bin/gcov gcov /usr/bin/gcov-13 130 && \
    update-alternatives --install /usr/bin/gcc-ar gcc-ar /usr/bin/gcc-ar-11 110 && \
    update-alternatives --install /usr/bin/gcc-ar gcc-ar /usr/bin/gcc-ar-13 130 && \
    update-alternatives --install /usr/bin/gcc-nm gcc-nm /usr/bin/gcc-nm-11 110 && \
    update-alternatives --install /usr/bin/gcc-nm gcc-nm /usr/bin/gcc-nm-13 130 && \
    update-alternatives --install /usr/bin/gcc-ranlib gcc-ranlib /usr/bin/gcc-ranlib-11 110 && \
    update-alternatives --install /usr/bin/gcc-ranlib gcc-ranlib /usr/bin/gcc-ranlib-13 130

# Install CMake 3.28.3 (Ubuntu 22.04 ships with older version)
# Provides better C++23 support and modern build features
RUN apt-get remove -y cmake cmake-data || true && \
    apt-get purge -y cmake cmake-data || true && \
    apt-get autoremove -y && \
    rm -rf /usr/bin/cmake* /usr/share/cmake* || true && \
    update-alternatives --remove-all cmake 2>/dev/null || true && \
    cd /tmp && \
    wget https://cmake.org/files/v3.28/cmake-3.28.3-linux-x86_64.tar.gz && \
    tar -xzf cmake-3.28.3-linux-x86_64.tar.gz && \
    cp -r cmake-3.28.3-linux-x86_64/bin/* /usr/local/bin/ && \
    cp -r cmake-3.28.3-linux-x86_64/share/cmake-3.28 /usr/local/share/ && \
    mkdir -p /usr/local/share/aclocal /usr/local/share/bash-completion/completions && \
    cp -r cmake-3.28.3-linux-x86_64/share/aclocal/* /usr/local/share/aclocal/ 2>/dev/null || true && \
    cp -r cmake-3.28.3-linux-x86_64/share/bash-completion/completions/* /usr/local/share/bash-completion/completions/ 2>/dev/null || true && \
    rm -rf cmake-3.28.3-linux-x86_64 cmake-3.28.3-linux-x86_64.tar.gz && \
    cmake --version

# ------------------------------------------------------------------------------
# ROS2 Humble Installation
# ------------------------------------------------------------------------------

# Add ROS2 repositories using TUNA mirror for faster access in China
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/ros2/ubuntu jammy main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null && \
    apt-get update -y

# Install ROS2 Humble Desktop Full with simulation and control packages
RUN apt-get update && DEBIAN_FRONTEND=noninteractive \
    apt-get install -y \
    ros-humble-desktop-full \
    ros-dev-tools \
    ros-humble-xacro \
    ros-humble-controller-manager \
    ros-humble-gazebo-ros-pkgs \
    ros-humble-gazebo-ros2-control \
    ros-humble-joint-state-publisher \
    ros-humble-joint-state-broadcaster \
    ros-humble-joy-teleop \
    python3-argcomplete \
    python3-colcon-common-extensions \
    python3-rosdep \
    python3-protobuf \
    python3-vcstool && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/*

# Create python symlink for compatibility with build scripts
RUN rm -rf /usr/bin/python && ln -s /usr/bin/python3 /usr/bin/python

# ------------------------------------------------------------------------------
# Python Development Tools
# ------------------------------------------------------------------------------

# Install Python packages for development, testing, and code quality
RUN pip3 install \
    click==8.0.3 \
    lark_parser==0.11.2 \
    pyyaml==6.0 \
    empy==3.3.4 \
    yq==2.13.0 \
    pylint==2.12.2 \
    jinja2==3.0.0 \
    selenium==3.141.0 \
    pcpp \
    codechecker \
    matplotlib \
    python-gitlab \
    pydot \
    xmlschema \
    pytest \
    pyecharts \
    cpplint \
    fabric \
    scp \
    autopep8 \
    debugpy \
    psutil \
    yamllint \
    PyQt5 \
    requests \
    webdriver_manager \
    colorama

# ------------------------------------------------------------------------------
# Additional Development Utilities
# ------------------------------------------------------------------------------

# Disable SSH host key checking for automated deployments
RUN sed -i 's/#   StrictHostKeyChecking.*/    StrictHostKeyChecking no/' /etc/ssh/ssh_config

# Install JFrog CLI for artifact management
RUN curl -fL https://install-cli.jfrog.io | sh

# Set clang-format-15 as default version
RUN update-alternatives --install /usr/bin/clang-format clang-format /usr/bin/clang-format-15 100

# Configure UTF-8 locale for internationalization support
ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
RUN locale-gen en_US.UTF-8

# Prioritize custom GCC in PATH
ENV PATH=/usr/local/gcc-13.1.0/bin:$PATH

# Increase Git buffer size for large repositories
RUN git config --global http.postBuffer 52428800

# Set Java home for JDK-dependent build tools
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/

# Install Bazel 7.5.0 build system
RUN wget https://github.com/bazelbuild/bazel/releases/download/7.5.0/bazel-7.5.0-linux-x86_64 -O /usr/local/bin/bazel && \
    chmod a+x /usr/local/bin/bazel

# ------------------------------------------------------------------------------
# NVIDIA Jetson Cross-Compilation Toolchain
# ------------------------------------------------------------------------------

# Download and extract NVIDIA L4T GCC toolchain for ARM64 Jetson targets (r36.3.0)
RUN mkdir -p /opt/nvidia/l4t-toolchain && \
    cd /opt/nvidia/l4t-toolchain && \
    wget https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v3.0/toolchain/aarch64--glibc--stable-2022.08-1.tar.bz2 && \
    tar -xvf /opt/nvidia/l4t-toolchain/aarch64--glibc--stable-2022.08-1.tar.bz2 -C /opt/nvidia/l4t-toolchain && \
    rm -f /opt/nvidia/l4t-toolchain/aarch64--glibc--stable-2022.08-1/usr && \
    rm -f /opt/nvidia/l4t-toolchain/aarch64--glibc--stable-2022.08-1.tar.bz2

# ------------------------------------------------------------------------------
# Cross-Compilation Sysroot Assembly
# ------------------------------------------------------------------------------
# Sysroot: Directory containing target system libraries/headers for linking
# ARM64 binaries on x86_64 host.
# ------------------------------------------------------------------------------

ARG SYSROOT=/opt/nvidia/orin_sysroot

# Copy ARM64 CUDA libraries from JetPack (cuBLAS, cuDNN, CUDA runtime, etc.)
COPY --from=jetpack_arm_source /usr/local/cuda-12.2 ${SYSROOT}/usr/local/cuda-12.2

# Replace ARM64 CUDA binaries with x86_64 versions for host execution
# nvcc and other tools run on host while generating ARM64 code
RUN rm -rf ${SYSROOT}/usr/local/cuda-12.2/bin
COPY --from=cuda_x86_source /usr/local/cuda-12.2/bin ${SYSROOT}/usr/local/cuda-12.2/bin

# Replace NVVM (CUDA IR optimizer) with x86_64 version for host execution
RUN rm -rf ${SYSROOT}/usr/local/cuda-12.2/nvvm/bin
COPY --from=cuda_x86_source /usr/local/cuda-12.2/nvvm/bin ${SYSROOT}/usr/local/cuda-12.2/nvvm/bin

# Copy ROS2 Humble ARM64 installation (libraries, headers, CMake configs)
COPY --from=ros_arm_source /opt/ros/humble ${SYSROOT}/opt/ros/humble

# Copy ROS2 ARM64 system dependencies for linking
# Essential for resolving ROS2 library dependencies during cross-compilation
COPY --from=ros_arm_source /usr/lib/aarch64-linux-gnu ${SYSROOT}/usr/lib/aarch64-linux-gnu
COPY --from=ros_arm_source /usr/include/aarch64-linux-gnu ${SYSROOT}/usr/include/aarch64-linux-gnu
COPY --from=ros_arm_source /usr/include ${SYSROOT}/usr/include

# Copy Jetson Multimedia API for hardware acceleration (video, camera, etc.)
COPY --from=jetpack_arm_source /usr/src/jetson_multimedia_api ${SYSROOT}/usr/src/jetson_multimedia_api

# Copy Jetson-specific ARM64 libraries and headers (NVENC, NVDEC, VPI, etc.)
COPY --from=jetpack_arm_source /usr/lib/aarch64-linux-gnu ${SYSROOT}/usr/lib/aarch64-linux-gnu
COPY --from=jetpack_arm_source /usr/include/aarch64-linux-gnu ${SYSROOT}/usr/include/aarch64-linux-gnu
COPY --from=jetpack_arm_source /usr/include ${SYSROOT}/usr/include
COPY --from=jetpack_arm_source /usr/lib/aarch64-linux-gnu/libvpi* ${SYSROOT}/usr/lib/aarch64-linux-gnu/
COPY --from=jetpack_arm_source /usr/lib/aarch64-linux-gnu/libnvvpi* ${SYSROOT}/usr/lib/aarch64-linux-gnu/

# Copy VPI (Vision Programming Interface) libraries
COPY --from=jetpack_arm_source /opt/nvidia/vpi3 ${SYSROOT}/opt/nvidia/vpi3

# Create symlinks for NVIDIA buffer surface libraries (GPU buffer management)
RUN find ${SYSROOT}/usr/lib/aarch64-linux-gnu -name "libnvbufsurface.so*" -exec ln -sf {} ${SYSROOT}/usr/lib/aarch64-linux-gnu/libnvbufsurface.so \; && \
    find ${SYSROOT}/usr/lib/aarch64-linux-gnu -name "libnvbufsurftransform.so*" -exec ln -sf {} ${SYSROOT}/usr/lib/aarch64-linux-gnu/libnvbufsurftransform.so \;

# Create symlink for ARM64 Python library on host system
RUN ln -s /opt/nvidia/orin_sysroot/usr/lib/aarch64-linux-gnu/libpython3.10.so /usr/lib/aarch64-linux-gnu/libpython3.10.so

# Create symlinks for VPI libraries and headers
RUN ln -sf ${SYSROOT}/opt/nvidia/vpi3/lib64/libvpi.so.3 ${SYSROOT}/usr/lib/aarch64-linux-gnu/libvpi.so && \
    ln -sf ${SYSROOT}/opt/nvidia/vpi3/include/vpi3 ${SYSROOT}/usr/include/vpi3 && \
    ln -sf ${SYSROOT}/opt/nvidia/vpi3/include/vpi ${SYSROOT}/usr/include/vpi

# ------------------------------------------------------------------------------
# ROS2 Environment Configuration
# ------------------------------------------------------------------------------

# Set ROS2 environment variables for tool operation
ENV ROS_DISTRO=humble
ENV AMENT_PREFIX_PATH=/opt/ros/humble
ENV COLCON_PREFIX_PATH=/opt/ros/humble
ENV ROS_PYTHON_VERSION=3
ENV ROS_VERSION=2

# ------------------------------------------------------------------------------
# Non-Root User Setup
# ------------------------------------------------------------------------------

# Create 'vscode' user with sudo privileges (UID/GID 1000 for host compatibility)
RUN groupadd --gid 1000 vscode \
    && useradd --uid 1000 --gid vscode --shell /bin/bash --create-home vscode \
    && echo "vscode ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/vscode \
    && chmod 0440 /etc/sudoers.d/vscode

USER vscode

# ------------------------------------------------------------------------------
# Shell Enhancement (Zsh Configuration)
# ------------------------------------------------------------------------------

# Install Oh My Zsh framework
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

# Install fzf (fuzzy finder) for interactive command/file searching
RUN wget https://github.com/junegunn/fzf/archive/refs/heads/master.zip -O /tmp/fzf.zip && \
  unzip /tmp/fzf.zip -d ~/.fzf && \
  mv ~/.fzf/fzf-master/* ~/.fzf/ && \
  rm -rf ~/.fzf/fzf-master && \
  ~/.fzf/install --all && \
  rm -f /tmp/fzf.zip && \
  echo "export FZF_DEFAULT_COMMAND='ag --hidden --ignore .git -l -g \"\"'" >>/home/vscode/.zshrc && \
  echo "export FZF_DEFAULT_OPTS=\"--height 40% --layout=reverse --preview '(highlight -O ansi {} || cat {}) 2> /dev/null | head -700'\"" >>/home/vscode/.zshrc

# Install zsh-autosuggestions for history-based command suggestions
RUN wget https://github.com/zsh-users/zsh-autosuggestions/archive/refs/heads/master.zip -O /tmp/zsh-autosuggestions.zip && \
  mkdir -p ~/.zsh && \
  unzip /tmp/zsh-autosuggestions.zip -d ~/.zsh && \
  mv ~/.zsh/zsh-autosuggestions-master ~/.zsh/zsh-autosuggestions && \
  rm -f /tmp/zsh-autosuggestions.zip && \
  echo "source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh" >>/home/vscode/.zshrc

# Install zsh-syntax-highlighting for real-time command validation
RUN wget https://github.com/zsh-users/zsh-syntax-highlighting/archive/refs/heads/master.zip -O /tmp/zsh-syntax-highlighting.zip && \
  mkdir -p ~/.zsh && \
  unzip /tmp/zsh-syntax-highlighting.zip -d ~/.zsh && \
  mv ~/.zsh/zsh-syntax-highlighting-master ~/.zsh/zsh-syntax-highlighting && \
  rm -f /tmp/zsh-syntax-highlighting.zip && \
  echo "source ~/.zsh/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >>/home/vscode/.zshrc

# Install autojump for quick directory navigation
RUN wget https://github.com/wting/autojump/archive/refs/heads/master.zip -O /tmp/autojump.zip && \
  mkdir -p ~/.zsh && \
  unzip /tmp/autojump.zip -d ~/.zsh && \
  cd ~/.zsh/autojump-master && \
  SHELL="zsh" python3 install.py && \
  rm -f /tmp/autojump.zip && \
  echo "[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh" >>/home/vscode/.zshrc

# Configure Zsh theme
RUN sed -i 's/^ZSH_THEME=.*/ZSH_THEME="robbyrussell"/' /home/vscode/.zshrc

# Auto-source ROS2 environment in shells
RUN echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc && \
    echo "source /opt/ros/humble/setup.zsh" >> ~/.zshrc

# Set Zsh as default shell
RUN sudo chsh -s /bin/zsh vscode

# ------------------------------------------------------------------------------
# Finalization
# ------------------------------------------------------------------------------

# Copy initialization utility scripts
COPY setup-init /usr/local/bin/
RUN sudo chmod a+x /usr/local/bin/setup-init

WORKDIR /workspaces

# Keep container running for interactive development
CMD [ "sleep", "infinity" ]
