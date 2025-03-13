# Airgap Deployment
This is the detailed explanation of air-gap deployment.

## 1. Offline Cache
This path use `download-to-cache` command, and the models will be cached under `$NIM_CACHE_PATH/ngc/hub` folder.
```shell
download-to-cache --profile PROFILE_HASH`
#NIM_CACHE_PATH=/opt/nim/.cache
ls /opt/nim/.cache/ngc/hub/models--nim--meta--llama-3.2-3b-instruct/
total 20
drwxr-xr-x 5 root   root   4096 Mar 13 10:26 ..
drwxr-xr-x 3 root   root   4096 Mar 13 10:26 snapshots
drwxr-xr-x 2 root   root   4096 Mar 13 10:26 refs
drwxr-xr-x 5 root   root   4096 Mar 13 10:26 .
drwxr-xr-x 2 root   root   4096 Mar 13 10:28 blobs
```
Host the model by setting `NIM_MODEL_PROFILE` environment variable. Without setting it, system will choose the best profile as a standard NIM startup procedure. (May need to download the model if not in cache)
```shell
 -e NIM_MODEL_PROFILE=PROFILE_HASH
```

## 2. Model Store
It uses `create-model-store` command the download the model to a specific path, and the model can be hosted as a HuggingFace model or TensorRT engine model.
```shell
create-model-store -p PROFILE_HASH --more-store /opt/nim/.cache/local_store/
ll /opt/nim/.cache/local_store/
total 6283988
drwxrwxrwx 57 root   root       4096 Mar 12 17:11 ..
-rw-r--r--  1 root   root        296 Mar 13 10:30 special_tokens_map.json
-rw-r--r--  1 root   root      35876 Mar 13 10:30 README.md
-rw-r--r--  1 root   root       6021 Mar 13 10:30 USE_POLICY.md
-rw-r--r--  1 root   root      54528 Mar 13 10:30 tokenizer_config.json
-rw-r--r--  1 root   root 4965799096 Mar 13 10:30 model-00001-of-00002.safetensors
-rw-r--r--  1 root   root       7712 Mar 13 10:30 LICENSE.txt
-rw-r--r--  1 root   root        189 Mar 13 10:30 generation_config.json
-rw-r--r--  1 root   root 1459729952 Mar 13 10:30 model-00002-of-00002.safetensors
-rw-r--r--  1 root   root        878 Mar 13 10:30 config.json
-rw-r--r--  1 root   root        147 Mar 13 10:30 tool_use_config.json
-rw-r--r--  1 root   root    9085657 Mar 13 10:30 tokenizer.json
-rw-r--r--  1 root   root      20919 Mar 13 10:30 model.safetensors.index.json
-rw-r--r--  1 root   root       1040 Mar 13 10:30 checksums.blake3
drwxr-xr-x  3 khuang root       4096 Mar 13 10:30 .
```
Host the model by setting `NIM_MODEL_NAME` environment variable.
```shell
 -e NIM_MODEL_NAME="/opt/nim/.cache/local_store/
```
