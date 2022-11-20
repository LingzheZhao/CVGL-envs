FROM harbor.cvgl.lab/nvidia/cudagl:11.4.2-devel-ubuntu20.04
ARG PYTHON_VERSION=3.8.15
ARG DEBIAN_FRONTEND=noninteractive
ARG OPENCV_VERSION=4.6.0
ARG OPENEXR_VERSION=3.1.5
ENV TZ=Asia/Shanghai LANG=C.UTF-8 LC_ALL=C.UTF-8 PIP_NO_CACHE_DIR=1 TORCH_CUDA_ARCH_LIST="8.6" MAX_JOBS=48

RUN sed -i  "s/archive.ubuntu.com/mirrors.ustc.edu.cn/g" /etc/apt/sources.list &&\
    sed -i  "s/security.ubuntu.com/mirrors.ustc.edu.cn/g" /etc/apt/sources.list &&\
    rm -f /etc/apt/sources.list.d/* &&\
    apt-get update && apt-get upgrade -y &&\
    apt-get install -y --no-install-recommends \
        autoconf automake autotools-dev build-essential ca-certificates \
        make cmake ninja-build yasm pkg-config checkinstall doxygen gdb \
        gcc gcc-multilib g++ g++-multilib gfortran \
        git nano curl wget unzip p7zip-full net-tools cloc htop iftop iotop ffmpeg \
        openssh-server openssh-client software-properties-common \
        libatlas-base-dev libboost-all-dev libcereal-dev libcgal-dev \
        libcxsparse3 libeigen3-dev liblapack-dev liblapacke-dev \
        metis libmetis-dev libopenblas-dev libsuitesparse-dev \
        libgtk2.0-dev libgtk-3-dev libglew-dev libgl1-mesa-dev libgl1-mesa-glx mesa-utils \
        '^libxcb.*-dev' libx11-xcb-dev libglu1-mesa-dev libxrender-dev \
        libxi-dev libxkbcommon-dev libxkbcommon-x11-dev \
        freeglut3-dev libassimp-dev libglfw3-dev libqt5opengl5-dev libxrandr-dev libxxf86vm-dev \
        libgflags-dev libgoogle-glog-dev libgtest-dev libprotobuf-dev protobuf-compiler \
        libgphoto2-dev libhdf5-dev libjpeg-dev libpng-dev libtiff-dev libtiff-dev\
        libavcodec-dev libavformat-dev libavutil-dev libavdevice-dev libavresample-dev \
        libpostproc-dev libpq-dev libswscale-dev \
        libtbb2 libtbb-dev libxine2-dev zlib1g-dev \
        libkrb5-dev libssl-dev libtool libyaml-dev libyaml-cpp-dev \
        libxvidcore-dev x264 libx264-dev libfaac-dev libmp3lame-dev libtheora-dev \
        libfaac-dev libmp3lame-dev libvorbis-dev \
        libopencore-amrnb-dev libopencore-amrwb-dev \
        libv4l-dev v4l-utils \
        libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev &&\
    rm /etc/ssh/ssh_host_ecdsa_key &&\
    rm /etc/ssh/ssh_host_ed25519_key &&\
    rm /etc/ssh/ssh_host_rsa_key &&\
    cp /etc/ssh/sshd_config /etc/ssh/sshd_config_bak &&\
    apt-get clean &&\
    rm -rf /var/lib/apt/lists/*

# Install Conda and Determined AI stuff
ENV PATH="/opt/conda/bin:${PATH}"
ENV PYTHONUNBUFFERED=1 PYTHONFAULTHANDLER=1 PYTHONHASHSEED=0
ENV JUPYTER_CONFIG_DIR=/run/determined/jupyter/config
ENV JUPYTER_DATA_DIR=/run/determined/jupyter/data
ENV JUPYTER_RUNTIME_DIR=/run/determined/jupyter/runtime
COPY dockerfile_scripts /tmp/det_dockerfile_scripts
RUN /tmp/det_dockerfile_scripts/install_python.sh ${PYTHON_VERSION} &&\
    pip config set global.index-url https://mirrors.bfsu.edu.cn/pypi/web/simple &&\
    pip install -U pip &&\
    pip install determined && pip uninstall -y determined &&\
    pip install -r /tmp/det_dockerfile_scripts/notebook-requirements.txt &&\
    /tmp/det_dockerfile_scripts/add_det_nobody_user.sh &&\
    /tmp/det_dockerfile_scripts/install_libnss_determined.sh &&\
    rm -rf /tmp/*

# Install OpenEXR
RUN cd /tmp &&\
    wget https://github.com/AcademySoftwareFoundation/openexr/archive/refs/tags/v${OPENEXR_VERSION}.tar.gz &&\
    tar -xvzf v${OPENEXR_VERSION}.tar.gz &&\
    cd openexr-${OPENEXR_VERSION} &&\
    cmake -B build -GNinja &&\
    cmake --build build -t install &&\
    ldconfig && rm -rf /tmp/*

# Install OpenCV
RUN cd /tmp/ &&\
    wget https://github.com/opencv/opencv/archive/$OPENCV_VERSION.zip &&\
    unzip $OPENCV_VERSION.zip &&\
    rm $OPENCV_VERSION.zip &&\
    wget https://github.com/opencv/opencv_contrib/archive/$OPENCV_VERSION.zip &&\
    unzip ${OPENCV_VERSION}.zip &&\
    cd opencv-${OPENCV_VERSION} &&\
    mkdir build && cd build &&\
    cmake \
        -DCMAKE_CXX_STANDARD=17 \
        -DENABLE_CCACHE=ON \
        -DOPENCV_EXTRA_MODULES_PATH=/tmp/opencv_contrib-${OPENCV_VERSION}/modules \
        -DWITH_CUDA=ON \
        -DWITH_TBB=ON \
        -DWITH_V4L=ON \
        -DWITH_QT=OFF \
        -DWITH_GSTREAMER=ON \
        -DOPENCV_ENABLE_NONFREE=ON \
        -DOPENCV_PYTHON3_INSTALL_PATH=/opt/conda/lib/python3.8/site-packages \
        -DPYTHON3_EXECUTABLE=/opt/conda/bin/python \
        -DINSTALL_PYTHON_EXAMPLES=OFF \
        -DINSTALL_C_EXAMPLES=OFF \
        -DBUILD_EXAMPLES=OFF \
        -DBUILD_opencv_python2=OFF \
        -DBUILD_opencv_python3=ON \
        -DBUILD_JAVA=OFF \
        -DCUDA_ARCH_BIN=8.6 \
        -DCMAKE_BUILD_TYPE=RELEASE \
        -DCMAKE_INSTALL_PREFIX=/usr/local \
        .. &&\
    make -j &&\
    make install &&\
    ldconfig && rm -rf /tmp/*

COPY .condarc /opt/conda/.condarc
RUN conda install pytorch torchvision torchaudio pytorch-cuda=11.7 -c pytorch -c nvidia
# Install lietorch
COPY lietorch-sm86.diff /tmp
RUN cd /tmp &&\
    git clone --recursive https://github.com/princeton-vl/lietorch &&\
    cd /tmp/lietorch &&\
    git apply /tmp/lietorch-sm86.diff &&\
    MAX_JOBS=32 /opt/conda/bin/python setup.py install &&\
    rm -rf /tmp/*
COPY ./environment.yaml /tmp/environment.yaml
RUN conda env update -n base -f /tmp/environment.yaml