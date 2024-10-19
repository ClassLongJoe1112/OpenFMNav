FROM nvcr.io/nvidia/cudagl:11.3.0-devel-ubuntu20.04

ARG USERNAME=user
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# # Create the user
# # add user with default bash
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    && rm -rf /var/lib/apt/lists/*

ENV SHELL=/bin/bash
ENV DEBIAN_FRONTEND=noninteractive

# Install common tools
RUN apt-get update && apt-get install -y \
    bash-completion \
    ca-certificates \
    curl \
    git \
    git-extras \
    git-lfs \
    build-essential \
    htop \
    tree \
    net-tools \
    tmux \
    vim \
    wget \
    zip \
    unzip \
    python3-pip \
    mesa-utils-extra \
    # Libraries
    libjpeg-dev \
    libpng-dev \
    # libglfw3-dev \
    # libglm-dev \
    libx11-dev \
    libomp-dev \
    libegl1-mesa-dev \
    && rm -rf /var/lib/apt/lists/*

# Install conda
RUN curl -L -o ~/miniconda.sh -O  https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh  &&\
    chmod +x ~/miniconda.sh &&\
    ~/miniconda.sh -b -p /opt/conda &&\
    rm ~/miniconda.sh &&\
    /opt/conda/bin/conda install numpy pyyaml scipy ipython mkl mkl-include &&\
    /opt/conda/bin/conda clean -ya
ENV PATH /opt/conda/bin:$PATH

# Install cmake
RUN wget https://github.com/Kitware/CMake/releases/download/v3.14.0/cmake-3.14.0-Linux-x86_64.sh
RUN mkdir /opt/cmake
RUN sh /cmake-3.14.0-Linux-x86_64.sh --prefix=/opt/cmake --skip-license
RUN ln -s /opt/cmake/bin/cmake /usr/local/bin/cmake
RUN cmake --version

# Conda environment
RUN conda create -n habitat python=3.7 cmake=3.14.0 -y
RUN /bin/bash -c ". activate habitat; conda install --yes habitat-sim-challenge-2022 withbullet -c conda-forge -c aihabitat"

# Install PyTorch.
# Reference:
# - https://pytorch.org/get-started/previous-versions/#linux-and-windows-26
RUN conda install -y -n habitat -c pytorch -c conda-forge \
pytorch==1.11.0 \
torchvision==0.12.0 \
torchaudio==0.11.0 \
cudatoolkit=11.3

# Install OpenFMNav dependencies
# (scikit-image is down from 0.21.0 to 0.19.3 due to habitat-sim-challenge-2022 requires python 3.7)
RUN /bin/bash -c ". activate habitat; pip install \
scikit-fmm \
scikit-learn \
scikit-image==0.19.3 \ 
numpy>=1.20.2 \
ifcfg \
transformers \
openai==0.28.1 \
retry"

# Install habitat-lab
RUN git clone --branch challenge-2022 https://github.com/facebookresearch/habitat-lab.git
RUN /bin/bash -c ". activate habitat; cd habitat-lab; pip install -e ."

# Install Grounded-SAM
RUN git clone https://github.com/IDEA-Research/Grounded-Segment-Anything.git
RUN /bin/bash -c ". activate habitat; cd Grounded-Segment-Anything; \
python -m pip install --no-cache-dir -e segment_anything; \
python -m pip install --no-cache-dir wheel; \
python -m pip install --no-cache-dir --no-build-isolation -e GroundingDINO"

# Install Grounded-SAM dependencies
RUN /bin/bash -c ". activate habitat; pip install \
diffusers[torch] \
opencv-python \
pycocotools \ 
matplotlib \
onnxruntime \
onnx \
ipykernel \
scipy \
gradio"

# USER $USERNAME

# Change the owner of habitat-lab
RUN sudo chown user /habitat-lab && sudo chgrp user /habitat-lab

CMD ["/bin/bash"]