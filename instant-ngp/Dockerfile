FROM nvidia/cuda:11.8.0-devel-ubuntu20.04

ARG DEBIAN_FRONTEND=noninteractive
ENV COLMAP_VERSION=3.7
ENV CMAKE_VERSION=3.21.0
ENV PYTHON_VERSION=3.8.15
ENV OPENCV_VERSION=4.5.5.62
ENV CERES_SOLVER_VERSION=2.0.0

RUN echo "Installing apt packages..." \
    sed -i  "s/archive.ubuntu.com/mirrors.ustc.edu.cn/g" /etc/apt/sources.list &&\
    sed -i  "s/security.ubuntu.com/mirrors.ustc.edu.cn/g" /etc/apt/sources.list &&\
    rm -f /etc/apt/sources.list.d/* &&\
	&& apt -y update --no-install-recommends \
	&& apt -y install --no-install-recommends \
	git \
    g++ \
    curl \
	wget \
    make \
    cmake \
	ffmpeg \
	tk-dev \
    libtool \
    autoconf \
    automake \
    libnuma1 \
	libxi-dev \
	libc6-dev \
	libbz2-dev \
	libffi-dev \
	libomp-dev \
	libssl-dev \
    nfs-common \
    pkg-config \
	zlib1g-dev \
    daemontools \
    ninja-build \
	libcgal-dev \
	libgdbm-dev \
	libglew-dev \
    libkrb5-dev \
    libnuma-dev \
	python3-dev \
	python3-pip \
	qtbase5-dev \
	checkinstall \
	libglfw3-dev \
    autotools-dev \
	libeigen3-dev \
	libgflags-dev \
    libpmi2-0-dev \
	libxrandr-dev \
    openssh-server \
    openssh-client \
	libopenexr-dev \
	libsqlite3-dev \
	libxcursor-dev \
	build-essential \
    ca-certificates \
	libcgal-qt5-dev \
	libxinerama-dev \
	libboost-all-dev \
	libfreeimage-dev \
	libncursesw5-dev \
	libatlas-base-dev \
	libqt5opengl5-dev \
	libgoogle-glog-dev \
	libsuitesparse-dev \
	python3-setuptools \
	libreadline-gplv2-dev \
	&& apt autoremove -y \
	&& apt clean -y \
	&& export DEBIAN_FRONTEND=dialog \
    && rm /etc/ssh/ssh_host_ecdsa_key \
    && rm /etc/ssh/ssh_host_ed25519_key \
    && rm /etc/ssh/ssh_host_rsa_key \
    && cp /etc/ssh/sshd_config /etc/ssh/sshd_config_bak \
    && sed -i "s/^.*X11Forwarding.*$/X11Forwarding yes/" /etc/ssh/sshd_config \
    && sed -i "s/^.*X11UseLocalhost.*$/X11UseLocalhost no/" /etc/ssh/sshd_config \
    && grep "^X11UseLocalhost" /etc/ssh/sshd_config || echo "X11UseLocalhost no" >> /etc/ssh/sshd_config \
    apt-get clean &&\
    rm -rf /var/lib/apt/lists/*

# Install Conda and Determined AI stuff
COPY .condarc /opt/conda/.condarc
ENV PATH="/opt/conda/bin:${PATH}"
ENV PYTHONUNBUFFERED=1 PYTHONFAULTHANDLER=1 PYTHONHASHSEED=0
ENV JUPYTER_CONFIG_DIR=/run/determined/jupyter/config
ENV JUPYTER_DATA_DIR=/run/determined/jupyter/data
ENV JUPYTER_RUNTIME_DIR=/run/determined/jupyter/runtime
RUN /tmp/det_dockerfile_scripts/install_python.sh ${PYTHON_VERSION} &&\
    pip config set global.index-url https://mirrors.bfsu.edu.cn/pypi/web/simple &&\
    pip install determined && pip uninstall -y determined &&\
    pip install -r /tmp/det_dockerfile_scripts/notebook-requirements.txt &&\
    /tmp/det_dockerfile_scripts/add_det_nobody_user.sh &&\
    /tmp/det_dockerfile_scripts/install_libnss_determined.sh &&\
    rm -rf /tmp/*

COPY ./requirements.txt ./

RUN echo "Installing pip packages..." \
	&& python3 -m pip install -U pip \
	&& pip3 --no-cache-dir install -r ./requirements.txt \
	&& pip3 --no-cache-dir install cmake==${CMAKE_VERSION} opencv-python==${OPENCV_VERSION} \
	&& rm ./requirements.txt

RUN echo "Installing Ceres Solver ver. ${CERES_SOLVER_VERSION}..." \
	&& cd /opt \
	&& git clone https://github.com/ceres-solver/ceres-solver \
	&& cd ./ceres-solver \
	&& git checkout ${CERES_SOLVER_VERSION} \
	&& mkdir ./build \
	&& cd ./build \
	&& cmake ../ -DBUILD_TESTING=OFF -DBUILD_EXAMPLES=OFF \
	&& make -j \
	&& make install

RUN echo "Installing COLMAP ver. ${COLMAP_VERSION}..." \
	&& cd /opt \
	&& git clone https://github.com/colmap/colmap \
	&& cd ./colmap \
	&& git checkout ${COLMAP_VERSION} \
	&& mkdir ./build \
	&& cd ./build \
	&& cmake ../ \
	&& make -j \
	&& make install \
	&& colmap -h
