## 1. Model hosting with SGLang
NIM added SGLang backend support in NIM 1.7.x and we can use this container to host model using SGLang.
```shell
services:
  nim-1.7:
    image: nvcr.io/nim/deepseek-ai/deepseek-r1:1.7.1
    container_name: nim-1.7
    volumes:
      - /raid/models/:/opt/nim/.cache/
      - type: bind
        source: sglang-launch.sh
        target: /sglang-launch.sh
    ports:
      - 30000:30000 # Expose the port for SGLang server
    environment:
      - NVIDIA_VISIBLE_DEVICES=0
    entrypoint: bash /sglang-launch.sh # Use the custom launch script for SGLang
```
Here is the script for `sglang-launch.sh`. The key step is to activate virtual environment for SGLang, then start SGLang server normally
```shell
source /opt/nim/sglang/.venv/bin/activate 
python3 -m sglang.launch_server \
    --model-path /opt/nim/.cache/Llama-3.1-8B-Instruct \
    --host 0.0.0.0 \
    --port 30000
```