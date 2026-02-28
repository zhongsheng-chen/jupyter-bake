# jupyter-bakery 定制镜像构建系统

## 📖 项目简介

本项目基于 Jupyter Docker Stacks，提供自动化构建流程，用于创建可定制的
Jupyter 镜像，支持多版本与多平台构建。

------------------------------------------------------------------------

## 📁 项目结构
```text
jupyter-bakery/
├── images/
│ ├── docker-stacks-foundation/ # 基础镜像层
│ │ ├── Dockerfile
│ │ ├── fix-permissions # 权限修复脚本
│ │ ├── initial-condarc # Conda 初始配置
│ │ ├── run-hooks.sh # 钩子脚本
│ │ ├── start.sh # 启动脚本
│ │ └── 10activate-conda-env.sh # Conda 环境激活
│ │
│ ├── base-notebook/ # Jupyter 基础层
│ │ ├── Dockerfile
│ │ ├── start-notebook.py
│ │ ├── start-notebook.sh
│ │ ├── start-singleuser.py
│ │ ├── start-singleuser.sh
│ │ ├── jupyter_server_config.py
│ │ └── docker_healthcheck.py
│ │
│ └── jupyter-custom-image/ # 定制层（用户自定义）
│ └── Dockerfile
│
├── scripts/
│ └── build.sh
│
├── docker-bake.hcl # Docker Bake 配置文件
├── docker-compose.yml
├── .env
├── requirements.txt
└── README.md
```
------------------------------------------------------------------------

## 🏗️ 镜像分层

``` mermaid
graph LR
    A[Ubuntu] --> B[Foundation]
    B --> C[Base Notebook]
    C --> D[Custom Image]
```

------------------------------------------------------------------------

## 🚀 快速开始

使用自动化脚本（推荐）

``` bash
# 构建默认版本（Ubuntu 22.04 + Python 3.10）
./build.sh

# 构建指定版本
./build.sh -m 3.9 -p 3.9.21 -u 20.04

# 构建并推送到 Docker Hub
./build.sh --push

# 构建单平台（amd64）
./build.sh --platforms linux/amd64
```

------------------------------------------------------------------------

## 🐳 多平台手动构建示例

### 1️⃣ 构建 foundation（Ubuntu 20.04 + Python 3.9）

``` bash
docker buildx build --platform linux/amd64 \
  -t zhongshengchen/docker-stacks-foundation:ubuntu20.04-py3.9 \
  --build-arg ROOT_IMAGE=ubuntu:20.04 \
  --build-arg PYTHON_VERSION=3.9 \
  --push \
  ./images/docker-stacks-foundation
```

### 2️⃣ 构建 base-notebook

``` bash
docker buildx build --platform linux/amd64 \
  -t zhongshengchen/base-notebook:ubuntu20.04-py3.9 \
  --build-arg BASE_IMAGE=zhongshengchen/docker-stacks-foundation:ubuntu20.04-py3.9 \
  --push \
  ./images/base-notebook
```

### 3️⃣ 构建 jupyter-custom-image

``` bash
docker buildx build --platform linux/amd64 \
  -t zhongshengchen/jupyter-custom:python-3.9.21-ubuntu20.04 \
  --build-arg BASE_IMAGE=zhongshengchen/base-notebook:ubuntu20.04-py3.9 \
  --build-arg PYTHON_VERSION=3.9.21 \
  --build-arg UBUNTU_VERSION=20.04 \
  -f images/jupyter-custom-image/Dockerfile \
  --push .
```

------------------------------------------------------------------------

## 🧪 启动测试

``` bash
docker run --rm zhongshengchen/jupyter-custom:python-3.9.21-ubuntu20.04 python --version
```

``` bash
docker run -it --rm -p 8888:8888 \
  zhongshengchen/jupyter-custom:python-3.9.21-ubuntu20.04 \
  start-notebook.sh
```

``` bash
docker run --rm zhongshengchen/jupyter-custom:python-3.9.21-ubuntu20.04 \
  python -c "import numpy, pandas, matplotlib; print('All good!')"
```

------------------------------------------------------------------------

## 📄 许可证

MIT License © 2024 Zhongsheng Chen

如果这个项目对你有帮助，请给个星！ ⭐
