FROM nvidia/vulkan:1.2.133-450

RUN rm /etc/apt/sources.list.d/cuda.list
RUN apt-key del 7fa2af80
RUN apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/3bf863cc.pub
RUN apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu2004/x86_64/7fa2af80.pub

RUN apt-get update
RUN apt-get install -yq tzdata && \
    ln -fs /usr/share/zoneinfo/America/New_York /etc/localtime && \
    dpkg-reconfigure -f noninteractive tzdata
RUN apt-get install -y \
        build-essential \
        gcc \
        g++ \
        python3.9 \
        cmake \
        ffmpeg \
        libx11-dev \
        libeigen3-dev \
        libusb-1.0-0-dev \
        libudev-dev \
        libv4l-dev \
        libjpeg-dev \
        libhidapi-dev \
        libopenhmd-dev \
        libxcb-randr0-dev \
        libbluetooth-dev \
        glslang-tools 
RUN apt-get install -y git
RUN git clone https://github.com/widiba03304/monado
WORKDIR /monado
RUN mkdir build && \
    mkdir install && \
    cd build && \
    cmake -DCMAKE_BUILD_TYPE=Debug -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=../install .. && \
    cmake --build . && \
    cmake --build . --target install
