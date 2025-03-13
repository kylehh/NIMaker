# Model Hosting
NIM container are model based. Each container is built based on NIM base container, and add model specific manifest files on top of it. So you can actually host different models with one NIM container.
## 1. Host Another NIM supported model
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

## 2. Host a HuggingFace model
As long as a huggingface model is supported by vLLM, the model can be directly hosted by NIM  
```
# Mount HF weight and assign the path to NIM_MODEL_NAME
export IMG_NAME="nvcr.io/nim/meta/llama-3.3-70b-instruct:latest"
docker run -it --rm \
  --gpus '"device=4,5,6,7"' \
  --shm-size=16GB \
  -v "$LOCAL_CACHE:/opt/nim/.cache" \
  -e NIM_MODEL_NAME="/opt/nim/.cache/PATH_TO_HF_WEIGHT" \
  -u root \
  $IMG_NAME 
```
For example, Llama 3.2 1B was downloaded under `/opt/nim/.cache/Llama-3.2-1B-Instruct/` and it can be hosted in NIM with vLLM backend 
```sh
# git clone https://huggingface.co/meta-llama/Llama-3.2-1B-Instruct
ls -lrt /opt/nim/.cache/Llama-3.2-1B-Instruct/
total 2422732
-rw-rw-r-- 1 1008 1010       7712 Oct 31 15:49 LICENSE.txt
-rw-rw-r-- 1 1008 1010        189 Oct 31 15:49 generation_config.json
-rw-rw-r-- 1 1008 1010        877 Oct 31 15:49 config.json
-rw-rw-r-- 1 1008 1010       6021 Oct 31 15:49 USE_POLICY.md
-rw-rw-r-- 1 1008 1010      41742 Oct 31 15:49 README.md
-rw-rw-r-- 1 1008 1010        296 Oct 31 15:49 special_tokens_map.json
-rw-rw-r-- 1 1008 1010      54528 Oct 31 15:49 tokenizer_config.json
-rw-rw-r-- 1 1008 1010    9085657 Oct 31 15:49 tokenizer.json
drwxrwxr-x 2 1008 1010       4096 Oct 31 15:50 original
-rw-rw-r-- 1 1008 1010 2471645608 Oct 31 15:50 model.safetensors
```
## 3. Host a JIT compiled TensorRT-LLM engine
This is similar to host a Huggingface model except that the model file is a compiled TensorRT-LLM Engine.  
```shell
# Mount TRTLLM engine and assign the path to NIM_MODEL_NAME
export IMG_NAME="nvcr.io/nim/meta/llama-3.3-70b-instruct:latest"
docker run -it --rm \
  --gpus '"device=4,5,6,7"' \
  --shm-size=16GB \
  -v "$LOCAL_CACHE:/opt/nim/.cache" \
  -e NIM_MODEL_NAME="/opt/nim/.cache/PATH_TO_TRTLLM_ENGINE" \
  -u root \
  $IMG_NAME 
```
and this is what we have under the TensorRT-LLM engine path
```shell
ls -lrt /opt/nim/.cache/OPT-llama32-1b
total 8948
-rw-rw-r-- 1 1008 root     189 Oct 31 08:49 generation_config.json
-rw-rw-r-- 1 1008 root     877 Oct 31 08:49 config.json
-rw-rw-r-- 1 1008 root     296 Oct 31 08:49 special_tokens_map.json
-rw-rw-r-- 1 1008 root 9085657 Oct 31 08:49 tokenizer.json
-rw-rw-r-- 1 1008 root   54528 Oct 31 08:49 tokenizer_config.json
drwxr-xr-x 2 1008 root    4096 Mar 12 16:04 trtllm_engine
## Engine files are located under trtllm_engine folder
ls -lrt /opt/nim/.cache/OPT-llama32-1b/trtllm_engine/
total 3004156
-rw-r--r-- 1 1008 root       5952 Mar 12 16:04 config.json
-rw-r--r-- 1 1008 root 3076240540 Mar 12 16:04 rank0.engine
```
You can use `nim-optimize` tool to create the TRTLLM engine, by supplying a config yaml file.
```shell
# Here is a simple example of config yaml file
# cat opt_llama32_1b.yaml
# dtype: float16
# tp_size: 1
# max_seq_len: 8192
MODEL_CHECKPOINT="Llama-3.2-1B-Instruct"
OUTPUT_MODEL="OPT-llama32-1b"
nim-optimize --model_dir ${MODEL_CHECKPOINT} \
 --build_config opt_llama32_1b.yaml \
 --output_engine_dir ${OUTPUT_MODEL}
```

## 4. Host a Finetuned Model
A finetuned model can be considered as a HuggingFace model, so the step 2 works. Another approach is to following instrctions [here](https://docs.nvidia.com/nim/large-language-models/latest/ft-support.html). It will create a JIT engine and host it. We use `NIM_FT_MODEL` to specify model weight path and use `NIM_CUSTOM_MODEL_NAME` to cache the compiled engine. 
```shell
export IMG_NAME="nvcr.io/nim/meta/llama-3.3-70b-instruct:latest"
docker run -it --rm \
  --gpus '"device=4,5,6,7"' \
  --shm-size=16GB \
  -v "$LOCAL_CACHE:/opt/nim/.cache" \
  -e NIM_FT_MODEL="/opt/nim/.cache/PATH_TO_MODEL_WEIGHT" \
  -e NIM_CUSTOM_MODEL_NAME="JIT_TRT_ENGINE" \
  -u root \
  $IMG_NAME 
```
The JIT engine will be saved under `$NIM_CACHE_FOLER/local_cache` with a manifest file. It can be listed by `list-model-profiles` command. 
```
ls /opt/nim/.cache/local_cache
-rw-r--r-- 1 1008 root 1126 Mar 12 17:00 local_manifest.yaml
drwxr-xr-x 3 1008 root 4096 Mar 12 17:00 JIT_TRTLLM_ENGINE
ls /opt/nim/.cache/local_cache/JIT_TRT_ENGINE
drwxr-xr-x 2 1008 root    4096 Mar 12 16:59 trtllm_engine
-rw-rw-r-- 1 1008 root     296 Mar 12 17:00 special_tokens_map.json
-rw-rw-r-- 1 1008 root     189 Mar 12 17:00 generation_config.json
-rw-rw-r-- 1 1008 root 9085657 Mar 12 17:00 tokenizer.json
-rw-rw-r-- 1 1008 root   54528 Mar 12 17:00 tokenizer_config.json
-rw-rw-r-- 1 1008 root     877 Mar 12 17:00 config.json
```
Please be aware, directly supply a TensorRT-LLM engine to `NIM_FT_MODEL` are NOT supported. (NIM 1.6)
```shell
  File "/opt/nim/llm/.venv/lib/python3.12/site-packages/tensorrt_llm/models/model_weights_loader.py", line 148, in detect_format
    raise NotImplementedError(
NotImplementedError: Only safetensors/pickle/binary directories are supported.
```