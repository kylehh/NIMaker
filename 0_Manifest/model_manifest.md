# Model Manifest
Model manifest file contains information about both TensorRT-LLM and vLLM backend engines.

You can directly get the engine list by `list-model-profiles`
```
docker run --rm -it nvcr.io/nim/meta/llama-3.3-70b-instruct:1.5.2 list-model-profiles
```
or, you can directly access the model manifest file by   
```
docker run --rm -it nvcr.io/nim/meta/llama-3.3-70b-instruct:1.5.2 cat /etc/nim/config/model_manifest.yaml

or

cat /opt/nim/etc/default/model_manifest.yaml
```

Here are the examples of 
- vLLM Engine  
List of NGC version of HuggingFace model artifact files.
```
- id: c84b2a0...
  tags:
    feat_lora: 'false'
    llm_engine: vllm
    pp: '1'
    precision: bf16
    tp: '4'
  workspace:
    files:
      LICENSE:
        uri: ngc://nim/meta/llama-3.3-70b-instruct:hf-5825c91?file=LICENSE
      README.md:
        uri: ngc://nim/meta/llama-3.3-70b-instruct:hf-5825c91?file=README.md
      USE_POLICY.md:
        uri: ngc://nim/meta/llama-3.3-70b-instruct:hf-5825c91?file=USE_POLICY.md
      config.json:
        uri: ngc://nim/meta/llama-3.3-70b-instruct:hf-5825c91?file=config.json
      ...
```
- TensorRT-LLM Buildable Engine  
It's similar to vLLM, just list the HuggingFace Files
```
- id: 1d7b60...
  tags:
    feat_lora: 'false'
    llm_engine: tensorrt_llm
    pp: '1'
    precision: bf16
    tp: '8'
    trtllm_buildable: 'true'
  workspace:
    files:
      LICENSE:
        uri: ngc://nim/meta/llama-3.3-70b-instruct:hf-5825c91?file=LICENSE
      README.md:
        uri: ngc://nim/meta/llama-3.3-70b-instruct:hf-5825c91?file=README.md
      USE_POLICY.md:
        uri: ngc://nim/meta/llama-3.3-70b-instruct:hf-5825c91?file=USE_POLICY.md
      config.json:
        uri: ngc://nim/meta/llama-3.3-70b-instruct:hf-5825c91?file=config.json
```      
- TensorRT-LLM Engine  
These are pre-build TensorRT Optimized Engines
```
- id: ee0d01...
  tags:
    gpu: A100
    gpu_device: 20b2:10de
    llm_engine: tensorrt_llm
    number_of_gpus: '1'
    pp: '1'
    precision: bf16
    profile: throughput
    tp: '1'
  workspace:
    files:
      LICENSE:
        uri: ngc://nim/qwen/qwen-2.5-7b-instruct:hf-bb46c15-jet?file=LICENSE
      trtllm_engine/rank0.engine:
        uri: ngc://nim/qwen/qwen-2.5-7b-instruct:a100x1-throughput-precision.bf16-udrspesh5q?file=rank0.engine
```