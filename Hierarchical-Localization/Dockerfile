FROM colmap/colmap:latest
ARG PYTHON_VERSION=3.8.15
ARG DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Shanghai LANG=C.UTF-8 LC_ALL=C.UTF-8 PIP_NO_CACHE_DIR=1
RUN rm /etc/apt/sources.list.d/cuda.list &&\
    rm /etc/apt/sources.list.d/nvidia-ml.list &&\
    sed -i "s@http://.*archive.ubuntu.com@http://mirrors.bfsu.edu.cn@g" /etc/apt/sources.list &&\
    sed -i "s@http://.*security.ubuntu.com@http://mirrors.bfsu.edu.cn@g" /etc/apt/sources.list &&\
    apt-get update -y &&\
    apt-get install -y unzip wget software-properties-common &&\
    add-apt-repository ppa:deadsnakes/ppa && \
    apt-get -y update && \
    apt-get install -y python${PYTHON_VERSION} &&\
    wget https://bootstrap.pypa.io/get-pip.py && python${PYTHON_VERSION} get-pip.py &&\
    update-alternatives --install /usr/bin/python3 python3 /usr/bin/python${PYTHON_VERSION} 1 &&\
    apt-get clean
COPY . /app
WORKDIR /app
RUN pip3 install --upgrade pip &&\
    pip3 install -r requirements.txt
RUN pip3 install jupyterlab ipywidgets
