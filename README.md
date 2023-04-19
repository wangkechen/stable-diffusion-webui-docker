# Stable Diffusion WebUI Docker

Easily and quickly run [stable-diffsion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) with [ControlNet](https://github.com/Mikubill/sd-webui-controlnet) in Docker.

## Requirements

* Docker
* Docker Compose

## Setup

```bash
$ cp .env.example .env
```

Edit your environment,

```bash
$ cat .env

TZ=Etc/UTC                   # timezone
PORT=7861                    # port mapping
VOLUME=data                  # docker volume storage mount point, eg. ./data to /data
DOCKERFILE=Dockerfile        # Specify Dockerfile
```

If you modified the value of **VOLUME**, for example from **VOLUME=data** to **VOLUME=volume**, please follow these steps as well,

```bash
$ cat .env

...
VOLUME=volume   # docker volume storage mount point, eg. ./data to /data
```

```bash
$ mv data volume
```

## Pre-download models and etc.

Here are some models for reference,

```bash
$ cat data/enhancing.sh

#!/usr/bin/env bash

declare -A arr

# models
arr["https://huggingface.co/prompthero/openjourney/resolve/main/mdjrny-v4.safetensors"]="models/Stable-diffusion"
arr+=(["https://huggingface.co/prompthero/openjourney-v2/resolve/main/openjourney-v2.ckpt"]="models/Stable-diffusion")
arr+=(["https://huggingface.co/ckpt/anything-v4.5-vae-swapped/resolve/main/anything-v4.5-vae-swapped.safetensors"]="models/Stable-diffusion")
arr+=(["https://huggingface.co/dreamlike-art/dreamlike-photoreal-2.0/resolve/main/dreamlike-photoreal-2.0.safetensors"]="models/Stable-diffusion")
arr+=(["https://huggingface.co/nuigurumi/basil_mix/resolve/main/Basil_mix_fixed.safetensors"]="models/Stable-diffusion")
arr+=(["https://huggingface.co/swl-models/chilloutmix-ni/resolve/main/chilloutmix-Ni-ema-fp32.safetensors"]="models/Stable-diffusion")
# vae
arr+=(["https://huggingface.co/stabilityai/sd-vae-ft-mse-original/resolve/main/vae-ft-mse-840000-ema-pruned.safetensors"]="models/VAE")
# embeddings
arr+=(["https://huggingface.co/datasets/gsdf/EasyNegative/resolve/main/EasyNegative.safetensors"]="embeddings")

...
```

Downloading ...

```bash
$ cd data ; bash enhancing.sh
```

These models, and others. will be linked in Docker once the container is successfully running.

## Installation

```bash
$ docker compose --profile cpu up -d     # for CPU

$ docker compose --profile gpu up -d     # for GPU
```

Please wait while the process completes.

If you want to track the progress,

```bash
$ docker compose --profile cpu logs -f   # for CPU

$ docker compose --profile gpu logs -f   # for GPU
```

## Usage

* Open your browser and navigate to **http://127.0.0.1:7861**. The default port specified in the **.env** file is **7861**.
* The saved images will be located in the **data/outputs** directory.
* Examples of prompts can be found in **data/prompts**.

## Restart, Stop or Deinstallation

```bash
$ docker compose --profile cpu restart   # for restart

$ docker compose --profile cpu stop      # for stop

$ docker compose --profile cpu down      # for deinstallation
```

## FAQ

### Q: Error message : "RuntimeError: mat1 and mat2 shapes cannot be multiplied (77x1024 and 768x320)"

![RuntimeError: mat1 and mat2 shapes cannot be multiplied](https://live.staticflickr.com/65535/52720126599_9910f655de_b.jpg "Stable Diffusion checkpoint")

When encountering the aforementioned error message, please select "**v1-5-pruned-emaonly.safetensors**" for the "**Stable Diffusion checkpoint**" to indicate that this operation is currently only supported by Stable Diffusion 1.5.

### Q: ERROR: Could not find a version that satisfies the requirement torch==1.13.1+cu117

If you come across the issue outlined in [stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/7166), a possible solution is to utilize Python 3.10, as described below:

Edit your environment, change **DOCKERFILE=Dockerfile** to **DOCKERFILE=Dockerfile.py310**.

```bash
$ cat .env

TZ=Etc/UTC                   # timezone
PORT=7861                    # port mapping
VOLUME=data                  # docker volume storage mount point, eg. ./data to /data
DOCKERFILE=Dockerfile.py310  # Specify Dockerfile
```

The next operation is as usual.

### Q: ERROR: docker: Error response from daemon: could not select device driver ““ with capabilities: [[gpu]]

docker在19.03版本之后，可以不用安装nvidia-docker，就能获得GPU的计算支持。

但是，直接在docker中使用GPU设备，出现报错如下：

docker: Error response from daemon: could not select device driver "" with capabilities: [[gpu]].

这个问题可以通过安装nvidia-container-toolkit来解决。

1、添加nvidia-docker的源

curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
如果不添加上面的源，可能会报如下错误：

E: Unable to locate package nvidia-container-toolkit

2、安装nvidia-container-toolkit

sudo apt-get install -y nvidia-container-toolkit
3、重启docker

sudo systemctl restart docker
问题解决。
————————————————
原文链接：https://blog.csdn.net/u013685264/article/details/123206768


### Q: Error message : AssertionError: Torch is not able to use GPU; add --skip-torch-cuda-test to COMMANDLINE_ARGS variable to disable this check

add --skip-torch-cuda-test in docker-compose.yml

BUILD_ARGS: --medvram --allow-code --enable-insecure-extension-access --api --listen --port ${PORT} --xformers --exit --skip-torch-cuda-test

and then:  docker compose --profile gpu up -d
