# Original Copyright 2021 DeepMind Technologies Limited
# Modifications Copyright 2022 Amazon.com, Inc. or its affiliates. All Rights Reserved.
# SPDX-License-Identifier: Apache-2.0

ARG CUDA=11.3.1
ARG UBUNTU_VERSION=20.04
FROM nvcr.io/nvidia/cuda:${CUDA}-cudnn8-runtime-ubuntu${UBUNTU_VERSION}

# FROM directive resets ARGS, so we specify again
ARG CUDA

# Use bash to support string substitution.
SHELL ["/bin/bash", "-c"]

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    cuda-command-line-tools-$(cut -f1,2 -d- <<< ${CUDA//./-}) \
    libcusparse-dev-11-3 \
    libcublas-dev-11-3 \
    libcusolver-dev-11-3 \
    git \
    wget \
    unzip \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

# Install AWS CLI
RUN wget -O "awscliv2.zip" "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
  && unzip awscliv2.zip \
  && ./aws/install \
  && rm awscliv2.zip

# Install Miniconda package manager.
RUN wget -q -P /tmp \
    https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh \ 
  && bash /tmp/Miniconda3-latest-Linux-x86_64.sh -b -p /opt/conda \
  && rm /tmp/Miniconda3-latest-Linux-x86_64.sh
ENV PATH="/opt/conda/bin:$PATH"
RUN conda update -n base -c conda-forge conda \
  && conda config --set ssl_verify no \
  && conda init bash

# Install ESMFold conda env
RUN git clone --depth 1 https://github.com/facebookresearch/esm.git /opt/esmfold
WORKDIR /opt/esmfold
RUN conda env update -n base --file environment.yml \
  && conda clean --all --yes

# Install OpenFold
RUN git clone --depth 1 https://github.com/aqlaboratory/openfold.git /opt/openfold
WORKDIR /opt/openfold
COPY setup.py .
RUN python setup.py install

# Install ESMFold code
WORKDIR /opt/esmfold
COPY run.sh /opt/esmfold/
ENV TORCH_HOME="/database/esmfold_params"
COPY esmfold_inference.py /opt/esmfold/scripts/
RUN pip install .

RUN chmod 755 run.sh
ENTRYPOINT ["bash", "run.sh"]
