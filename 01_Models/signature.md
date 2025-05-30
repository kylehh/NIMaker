# Sigiture verification for NIM container and models

## 1. Container Verification
- Software: cosign (Installation guide [here](https://docs.sigstore.dev/cosign/system_config/installation/))
- Public Key: https://api.ngc.nvidia.com/v2/catalog/containers/public-key
- Verification step:
```sh
# You do NOT need to download the public key nor the container
cosign verify --insecure-ignore-tlog \
       --key https://api.ngc.nvidia.com/v2/catalog/containers/public-key \
       nvcr.io/nim/meta/llama-3.2-3b-instruct:1.8.3
```

## 2. Models
- Software: model_signing
- Public key: https://api.ngc.nvidia.com/v2/catalog/models/public-key
    - You have to download it as `model_publickey.pem`, not like container verifiaction can use the link
- Signature download `result.sigstore`
    - `ngc registry model download-version-signature nim/meta/llama-3.2-1b-instruct:h200x1-throughput-fp8-ajzq-idegq`
- Model download
    - `ngc registry model download-version nim/meta/llama-3.2-1b-instruct:h200x1-throughput-fp8-ajzq-idegq`
- Verification 
```sh
model_signing verify certificate --signature result.sigstore \
     --certificate_chain ./models_publickey.pem \
     llama-3.2-1b-instruct_vh200x1-throughput-fp8-ajzq-idegq/
```