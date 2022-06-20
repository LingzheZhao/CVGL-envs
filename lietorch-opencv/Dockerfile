FROM determinedai/environments:cuda-11.3-pytorch-1.10-lightning-1.5-tf-2.8-deepspeed-0.5.10-gpu-0.18.2
ARG DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Shanghai LANG=C.UTF-8 LC_ALL=C.UTF-8 PIP_NO_CACHE_DIR=1

COPY lietorch-sm86.diff /tmp

RUN sed -i  "s/archive.ubuntu.com/mirrors.ustc.edu.cn/g" /etc/apt/sources.list &&\
    sed -i  "s/security.ubuntu.com/mirrors.ustc.edu.cn/g" /etc/apt/sources.list &&\
    apt-get update && apt-get upgrade -y &&\
    apt-get install -y --no-install-recommends \
        autoconf automake autotools-dev build-essential ca-certificates \
        make cmake yasm pkg-config gdb \
        gcc gcc-multilib g++ g++-multilib \
        libcxsparse3 libeigen3-dev liblapack-dev liblapacke-dev &&\
    apt-get clean &&\
    conda config --set show_channel_urls yes &&\
    conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/ &&\
    conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/main/ &&\
    conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/ &&\
    conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/msys2/ &&\
    conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/bioconda/ &&\
    conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/menpo/ &&\
    conda install -c conda-forge opencv &&\
    cd /tmp &&\
    git clone --recursive https://github.com/princeton-vl/lietorch &&\
    cd /tmp/lietorch &&\
    git apply /tmp/lietorch-sm86.diff &&\
    /opt/conda/bin/pip install -U numpy==1.22.2 &&\
    /opt/conda/bin/python setup.py install &&\
    rm -rf /tmp/*
