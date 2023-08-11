FROM nvidia/cuda:12.1.1-devel-ubuntu22.04

ARG PYVER=3.8

LABEL com.nvidia.volumes.needed="nvidia_driver"

RUN apt-get update && apt-get install -y --no-install-recommends \
        ocl-icd-libopencl1 \
        clinfo pkg-config && \
    rm -rf /var/lib/apt/lists/*

RUN mkdir -p /etc/OpenCL/vendors && \
    echo "libnvidia-opencl.so.1" > /etc/OpenCL/vendors/nvidia.icd

RUN echo "/usr/local/nvidia/lib" >> /etc/ld.so.conf.d/nvidia.conf && \
    echo "/usr/local/nvidia/lib64" >> /etc/ld.so.conf.d/nvidia.conf

ENV PATH /usr/local/nvidia/bin:${PATH}
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib:/usr/local/nvidia/lib64:${LD_LIBRARY_PATH}

# nvidia-container-runtime
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility
################################ end nvidia opencl driver ################################

# Some essentials
RUN apt-get update && \
    apt-get install -y wget make clinfo build-essential git libcurl4-openssl-dev \
    libssl-dev zlib1g-dev libcurl4-openssl-dev libssl-dev pciutils software-properties-common vim-tiny

ARG DEBIAN_FRONTEND=noninteractive

RUN add-apt-repository ppa:deadsnakes/ppa
RUN apt-get update && apt-get install -y python$PYVER python3-pip python$PYVER-distutils python$PYVER-dev
RUN touch /root/.bashrc
RUN echo "alias python='python$PYVER'" >> /root/.bashrc

# Fetch PCI IDs list to display proper GPU names
RUN update-pciids

WORKDIR /root

RUN apt-get install -y g++ cmake ninja-build llvm-15-dev libz-dev libglew-dev flex bison libfl-dev libboost-thread-dev libboost-filesystem-dev libzstd-dev
RUN git clone https://github.com/gpuocelot/gpuocelot.git && \
    cd gpuocelot && \
    git submodule init && \
    git submodule update && \
    cd ocelot && \
    mkdir build && \
    cd build && \
    cmake .. -G Ninja && \
    ninja && \
    ninja install

# Set up tinygrad
RUN git clone https://github.com/tinygrad/tinygrad.git


WORKDIR /root/tinygrad
ENV CUDA 1
# this gets stale so update when you run, but baking it in saves some startup time
RUN python$PYVER -m pip install -e '.[testing, cuda]' --extra-index-url https://download.pytorch.org/whl/cpu

