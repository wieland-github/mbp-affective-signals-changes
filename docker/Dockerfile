FROM ubuntu:18.04
RUN apt-get update && \
    apt-get install -y git wget ffmpeg libsm6 libxext6 && \
    wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    bash Miniconda3-latest-Linux-x86_64.sh -b -p /miniconda && \
    rm -rf Miniconda3-latest-Linux-x86_64.sh
WORKDIR /home/
RUN mkdir -p Notebooks rPPG-Data
ENV PATH=$PATH:/miniconda/bin
COPY . .



# Create and set up the conda environment from the existing yml file
RUN conda env create -f ./signal.yml && \
    conda clean -afy
# Clone the rPPG-Toolbox repository
RUN git clone https://github.com/ubicomplab/rPPG-Toolbox.git /home/rPPG-Toolbox
# Install PyTorch with the specific version required by rPPG-Toolbox
RUN conda run -n signal pip install torch==2.1.2+cu121 torchvision==0.16.2+cu121 torchaudio==2.1.2+cu121 --index-url https://download.pytorch.org/whl/cu121


RUN conda run -n signal pip install biosignalsnotebooks || \
    conda run -n signal pip install biosignalsnotebooks==0.5.0 || \
    conda run -n signal pip install biosignalsnotebooks==0.6.13 --no-deps || \
    echo "Failed to install biosignalsnotebooks, continuing anyway"

# Install the rPPG-Toolbox requirements
RUN conda run -n signal pip install typing-extensions>=4.0.0
# Install the requirements manually, skipping causal-conv1d
RUN cat /home/rPPG-Toolbox/requirements.txt | grep -v -E 'causal-conv1d|mamba-ssm' > /tmp/requirements.txt && \
    conda run -n signal pip install -r /tmp/requirements.txt
# Try to install causal-conv1d without version constraint
RUN conda run -n signal pip install causal-conv1d || echo "Failed to install causal-conv1d, continuing anyway"
# Install the Mamba tool
WORKDIR /home/rPPG-Toolbox/tools/mamba
RUN conda run -n signal python setup.py install || echo "Failed to install mamba, continuing anyway"


# Install PyQt5 for interactive plotting
RUN conda run -n signal pip install PyQt5
# Verify installation
RUN conda run -n signal python -c "import torch; print('PyTorch version:', torch.__version__); print('CUDA available:', torch.cuda.is_available())"


# Set up the environment activation
WORKDIR /home/
RUN echo "source activate signal" > ~/.bashrc
ENV PATH=$PATH:/miniconda/envs/signal/bin
EXPOSE 8888
CMD ["bash"]