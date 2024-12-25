FROM nvidia/cuda:12.6.3-cudnn-devel-ubuntu24.04

ENV DEBIAN_FRONTEND=noninteractive
ARG PYTHON_VERSION=3.10

RUN sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list \
    && sed -i s@/security.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list \
    && apt-get update -y && apt-get install -y git unzip git-lfs  \
    && git lfs install \
    && apt-get install -y python${PYTHON_VERSION} python${PYTHON_VERSION}-dev python${PYTHON_VERSION}-venv \
    && update-alternatives --install /usr/bin/python3 python3 /usr/bin/python${PYTHON_VERSION} 1 \
    && update-alternatives --set python3 /usr/bin/python${PYTHON_VERSION} \
    && ln -sf /usr/bin/python${PYTHON_VERSION}-config /usr/bin/python3-config \
    && curl -sS https://bootstrap.pypa.io/get-pip.py | python${PYTHON_VERSION} \
    && python3 --version && python3 -m pip --version

RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Workaround for CUDA compatibility issues
RUN ldconfig /usr/local/cuda-$(echo $CUDA_VERSION | cut -d. -f1,2)/compat/

WORKDIR /opt/CosyVoice

RUN git clone --recursive https://github.com/wonglyxng/CosyVoice.git

RUN cd CosyVoice && pip install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple/ --trusted-host=mirrors.aliyun.com
RUN cd CosyVoice/runtime/python/grpc && python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. cosyvoice.proto

# Set CUDA-related environment variables
ENV CUDA_HOME=/usr/local/cuda
ENV PATH=${CUDA_HOME}/bin:${PATH}
ENV LD_LIBRARY_PATH=${CUDA_HOME}/lib64:${LD_LIBRARY_PATH}

# Set CUDA architecture list
ARG TORCH_CUDA_ARCH_LIST="7.0 7.5 8.0 8.6+PTX"
ENV TORCH_CUDA_ARCH_LIST=${TORCH_CUDA_ARCH_LIST}