FROM tensorflow/tensorflow:2.16.1-gpu

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    python3-venv \
    git \
    sudo \
    && rm -rf /var/lib/apt/lists/*

RUN useradd -m vscode \
    && echo "vscode ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

RUN pip3 install --upgrade pip

RUN pip3 install \
    tensorflow==2.16.1 \
    jupyterlab \
    ipykernel \
    scikit-learn \
    pandas \
    numpy \
    matplotlib

USER vscode
WORKDIR /workspace