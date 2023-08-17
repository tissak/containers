# Import necessary base images
FROM runpod/stable-diffusion:models-1.0.0 as sd-models
FROM runpod/stable-diffusion-models:2.1 as hf-cache
FROM nvidia/cuda:11.8.0-devel-ubuntu22.04 as runtime

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Set working directory and environment variables
ENV SHELL=/bin/bash
ENV PYTHONUNBUFFERED=1
ENV DEBIAN_FRONTEND=noninteractive

WORKDIR /

# Set up system
RUN apt-get update --yes && \
    apt-get upgrade --yes && \
    apt install --yes --no-install-recommends git wget curl bash libgl1 software-properties-common openssh-server nginx rsync && \
    add-apt-repository ppa:deadsnakes/ppa && \
    apt install python3.10-dev python3.10-venv -y --no-install-recommends && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen

# Set up Python and pip
RUN ln -s /usr/bin/python3.10 /usr/bin/python && \
    rm /usr/bin/python3 && \
    ln -s /usr/bin/python3.10 /usr/bin/python3 && \
    curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && \
    python get-pip.py

# Install necessary Python packages
RUN pip install --upgrade --no-cache-dir pip
RUN pip install --upgrade --no-cache-dir torch==2.0.1+cu118 torchvision==0.15.2+cu118 torchaudio==2.0.2 --index-url https://download.pytorch.org/whl/cu118
RUN pip install --upgrade --no-cache-dir jupyterlab ipywidgets jupyter-archive jupyter_contrib_nbextensions triton xformers==0.0.18 gdown

RUN python -m venv /venv
ENV PATH="/venv/bin:$PATH"

# Install ComfyUI
RUN git clone https://github.com/comfyanonymous/ComfyUI.git && \
    cd /ComfyUI && \
    pip install -r requirements.txt

# NGINX Proxy
COPY --from=proxy nginx.conf /etc/nginx/nginx.conf
COPY --from=proxy readme.html /usr/share/nginx/html/readme.html
COPY README.md /usr/share/nginx/html/README.md

# Create necessary directories and copy necessary files
RUN set -e && mkdir -p /root/.cache/huggingface
# Convenience move the models folder to the root
RUN ln -s /workspace/ComfyUI/models /comfy-models
RUN ln -s /workspace/ComfyUI/custom_nodes /custom_nodes

COPY --from=hf-cache /root/.cache/huggingface /root/.cache/huggingface
COPY --from=sd-models /SDv1-5.ckpt /comfy-models/v1-5-pruned-emaonly.ckpt

# RUN wget https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0/resolve/main/sd_xl_base_1.0.safetensors -O /comfy-models/sd_xl_base_1.0.safetensors && \
#     wget https://huggingface.co/stabilityai/stable-diffusion-xl-refiner-1.0/resolve/main/sd_xl_refiner_1.0.safetensors -O /comfy-models/sd_xl_refiner_1.0.safetensors

RUN wget https://huggingface.co/XpucT/Deliberate/blob/main/Deliberate_v2.safetensors -O /comfy-models/checkpoints/Deliberate_v2.safetensors
RUN wget https://huggingface.co/digiplay/AbsoluteReality_v1.8.1/blob/main/absolutereality_v181.safetensors -O /comfy-models/checkpoints/absolutereality_v181.safetensors

# Custom ComfyUI Nodes
RUN git clone https://github.com/ltdrdata/ComfyUI-Impact-Pack.git /custom_nodes/ComfyUI-Impact-Pack
RUN git clone https://github.com/LucianoCirino/efficiency-nodes-comfyui.git /custom_nodes/efficiency-nodes-comfyui
RUN git clone https://github.com/space-nuko/ComfyUI-OpenPose-Editor.git /custom_nodes/ComfyUI-OpenPose-Editor
RUN git clone https://github.com/Fannovel16/comfy_controlnet_preprocessors.git /custom_nodes/comfy_controlnet_preprocessors

#Re-Install? TODO: Figure out why it says it is not installed.
RUN pip install torchvision


# Start Scripts
COPY pre_start.sh /pre_start.sh
COPY --from=scripts start.sh /
RUN chmod +x /start.sh

CMD [ "/start.sh" ]
