# Ascend 950 (A5) Docker Image Build and Usage Guide

## Overview
This guide provides step-by-step instructions for building and using the Ascend 950 (A5) Docker image for VeOmni framework. The A5 product uses the Ascend 910A chip and supports both x86_64 and ARM64 architectures.

> **Note**: For Ascend A2 (910B) Docker images, please refer to [Ascend A2 Docker Guide](../build_a2_docker.md).

## Prerequisites
- Docker installed on your system
- Access to Ascend 950 (A5) hardware accelerators
- Network access to pull the base image and install dependencies
- Proxy configuration (if required in your environment)

## Step 1: Pull the Base Image
First, pull the Huawei Ascend CANN base image for A5 (910A chip).

```bash
# for x86
docker pull --platform=amd64 swr.cn-south-1.myhuaweicloud.com/ascendhub/cann:9.0.0-910a-ubuntu22.04-py3.11

# for arm64
docker pull --platform=arm64 swr.cn-south-1.myhuaweicloud.com/ascendhub/cann:9.0.0-910a-ubuntu22.04-py3.11
```

> **Note**: Replace the base image tag with the CANN version provided by your A5 deployment environment.

## Step 2: Build the Custom Image
Build the VeOmni A5 image using the Dockerfile appropriate for your architecture.

```bash
# For x86 Architecture
docker build \
  --build-arg http_proxy=http://<user>:<pass>@<host>:<port> \
  --build-arg https_proxy=http://<user>:<pass>@<host>:<port> \
  --build-arg no_proxy=localhost,127.0.0.1 \
  -t ascend-a5-env:v1 \
  -f docker/ascend/Dockerfile.ascend_9.0.0_a5.x86 \
  .

# For ARM64 Architecture
docker build \
  --build-arg http_proxy=http://<user>:<pass>@<host>:<port> \
  --build-arg https_proxy=http://<user>:<pass>@<host>:<port> \
  --build-arg no_proxy=localhost,127.0.0.1 \
  -t ascend-a5-env:v1 \
  -f docker/ascend/Dockerfile.ascend_9.0.0_a5.arm \
  .
```

## Step 3: Run the Container

### Basic Container Start
```bash
docker run --runtime=runc -it \
  --ulimit nproc=65535 \
  --ulimit nofile=65535 \
  --device=/dev/davinci* \
  --device=/dev/davinci_manager \
  --device=/dev/devmm_svm \
  --device=/dev/hisi_hdc \
  -v /usr/local/Ascend/driver/lib64:/usr/local/Ascend/driver/lib64:ro \
  -v /usr/local/Ascend/driver/tools:/usr/local/Ascend/driver/tools:ro \
  -v /usr/local/Ascend/add-ons:/usr/local/Ascend/add-ons:ro \
  --name ascend-a5-container \
  ascend-a5-env:v1 \
  /bin/bash
```

## Important Notes for A5

### Operator Configuration
When training on A5, use the following operator configuration in your YAML:

```yaml
model:
  ops_implementation:
    attn_implementation: "sdpa"
    moe_implementation: "fused_npu"
    cross_entropy_loss_implementation: "npu"
    rms_norm_implementation: "npu"
    rotary_pos_emb_implementation: "npu"
    swiglu_mlp_implementation: "eager"
    load_balancing_loss_implementation: "eager"
    rms_norm_gated_implementation: "eager"
```

> **Note**: Do not use `flash_attention_2` or `liger_kernel` backends on A5. See [A5 Unsupported Features and Limitations](../a5_unsupported_features.md) for details.

### Model Size Limitations
A5 (Ascend 910A) has lower memory capacity than A2 (910B). Recommended maximum model sizes on A5:
- Dense models: up to 8B parameters (FSDP2)
- MoE models: not validated on A5; use A2/A3 for MoE training

For more information about A5 limitations, see [A5 Unsupported Features and Limitations](../a5_unsupported_features.md).
