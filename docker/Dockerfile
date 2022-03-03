# Define function directory (Lambda RIE)
ARG FUNCTION_DIR="/function"

# Define argments for mmdetection
ARG PYTORCH="1.6.0"
ARG CUDA="10.1"
ARG CUDNN="7"

# Base image (mmdetection)
FROM pytorch/pytorch:${PYTORCH}-cuda${CUDA}-cudnn${CUDNN}-devel

ENV TORCH_CUDA_ARCH_LIST="6.0 6.1 7.0+PTX"
ENV TORCH_NVCC_FLAGS="-Xfatbin -compress-all"
ENV CMAKE_PREFIX_PATH="$(dirname $(which conda))/../"

RUN apt-get update && \
    apt-get install -y ffmpeg libsm6 libxext6 git ninja-build libglib2.0-0 libsm6 libxrender-dev libxext6

# Install MMCV
RUN pip install --no-cache-dir --upgrade pip wheel setuptools
RUN pip install --no-cache-dir mmcv-full==1.3.17 -f https://download.openmmlab.com/mmcv/dist/cu101/torch1.6.0/index.html

# Install MMDetection
RUN conda clean --all
# RUN git clone https://github.com/open-mmlab/mmdetection.git /mmdetection
# Clone mmdetection repo in hayashi-lab-soma remote repo
RUN git clone -b bcc-app https://github.com/hayashi-lab-soma/mmdetection.git /mmdetection
WORKDIR /mmdetection
ENV FORCE_CUDA="1"
RUN pip install --no-cache-dir -r requirements/build.txt
RUN pip install --no-cache-dir -e .


# AWS Lambda Runtime Interface Client
# Install depends packages
RUN apt-get update && apt-get install -y \
  g++ \
  make \
  cmake \
  unzip \
  wget \
  libcurl4-openssl-dev \
  libsm6 \
  libxrender1 \
  libxtst6 

RUN apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Download sample checkpoint files
RUN wget https://download.openmmlab.com/mmdetection/v2.0/htc/htc_r50_fpn_1x_coco/htc_r50_fpn_1x_coco_20200317-7332cf16.pth -P /mmdetection/checkpoints
RUN wget https://download.openmmlab.com/mmdetection/v2.0/mask_rcnn/mask_rcnn_r50_fpn_1x_coco/mask_rcnn_r50_fpn_1x_coco_20200205-d4b0c5d6.pth -P /mmdetection/checkpoints


# Prepare to download original checkpoint files
RUN chmod +x /mmdetection/gdrive_download.sh
RUN ["/bin/bash", "-c", "source /mmdetection/gdrive_download.sh"]
RUN ["/bin/bash", "-c", "source /mmdetection/download_pth.sh"]


# COPY entry script and rie execute 
RUN cp /mmdetection/docker/entry_script.sh /entry_script.sh
RUN cp /mmdetection/docker/aws-lambda-rie-arm64 /usr/local/bin/aws-lambda-rie 
ENTRYPOINT [ "/entry_script.sh" ]


# Create function directory
ARG FUNCTION_DIR
RUN mkdir -p ${FUNCTION_DIR}

RUN pip install --upgrade pip && \
    pip install --target ${FUNCTION_DIR} awslambdaric
RUN pip install --upgrade pip && pip install boto3

# Copy function code
# "app.py" in local app directory -> ${FUNCTION_DIR}/app.py
ARG FUNCTION_DIR
COPY app/* ${FUNCTION_DIR}

# Set entry point (python module) for Lambda trigger
CMD ["app.hander"]
