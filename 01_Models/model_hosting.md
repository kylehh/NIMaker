# Model Hosting
NIM container are model based. Each container is built based on NIM base container, and add model specific manifest files on top of it. So you can actually host different models with one NIM container.
1. Host Another NIM supported model
You can copy the manifest file over and modify corresponding environment variables.   
```
# Host Mistral 7B model with Llama 3.1 8B container
# Download model_manifest.yaml from Mistral 7B container first
# and build a new container with following command

FROM nvcr.io/nim/meta/llama-3.1-8b-instruct:latest
COPY ./model_manifest_mistral7b.yaml /etc/nim/config/model_manifest.yaml
ENV NIM_SERVED_MODEL_NAME=mistralai/mistral-7b-instruct-v0.3
ENV NIM_MODEL_NAME=mistralai/mistral-7b-instruct-v0.3
```

  Or you can build a new container from the NIM base image.
This container is not public released though. 
```
FROM nvcr.io/nvstaging/nim/nim-llm:tag
COPY ./model_manifest_mistral7b.yaml /etc/nim/config/model_manifest.yaml
ENV NIM_SERVED_MODEL_NAME=mistralai/mistral-7b-instruct-v0.3
ENV NIM_MODEL_NAME=mistralai/mistral-7b-instruct-v0.3
CMD ["/opt/nim/start-server.sh"]
```

2. Host a HuggingFace model
As long as a huggingface model is supported by vLLM, the model can be directly hosted by NIM  
```
# Mount HF weight and assign the path to NIM_MODEL_NAME
export IMG_NAME="nvcr.io/nim/meta/llama-3.3-70b-instruct:latest"
docker run -it --rm --name=$CONTAINER_NAME \
  --gpus '"device=4,5,6,7"' \
  --shm-size=16GB \
  -v "$LOCAL_CACHE:/opt/nim/.cache" \
  -e NIM_MODEL_NAME="/opt/nim/.cache/PATH_TO_HF_WEIGHT" \
  -u root \
  $IMG_NAME  /bin/bash
```