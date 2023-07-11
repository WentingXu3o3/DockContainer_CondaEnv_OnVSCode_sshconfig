# DockContainer_CondaEnv_OnVSCode
INStruction for building container and Conda Environment on VsCode
# Docker Container
1. add a new folder in vscode as the workspace file for docker container
2. add a new folder under the directory with .devcontianer
3. add devcontainer.json file
```
{
    "name": "3dssg",
    "dockerComposeFile": "docker-compose.yml",
    "runServices": ["devcontainer"],
    "service": "devcontainer",
    "workspaceFolder": "/home/wenting/code/3DSSG",
    "customizations": {"vscode":{"extensions": [
        "ms-python.python",
        "esbenp.prettier-vscode",
        "eamodio.gitlens"
    ]}},
    "shutdownAction": "none"
   //当关闭vscode时docker container不会关掉 适用于下载或者跑长项目时
}
```
5. add docker-compose.yml
```
version:  "3.7"
services: 
  devcontainer:
    build: .
    image: 3dssg
    container_name: 3dssg
    volumes:
      - ..:/home/wenting/code/3DSSG
      - ~/.ssh:/root/.ssh:ro
      - ~/.gitconfig:/root/.gitconfig:ro
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
    entrypoint: bash
    stdin_open: true
    tty: true
```
7. add Dockerfile
can pip all new packages here.
```
FROM pytorch/pytorch:2.0.0-cuda11.7-cudnn8-devel
RUN apt-get update && apt-get install -y \
    git \
    libsparsehash-dev

RUN pip install --upgrade git+https://github.com/mit-han-lab/torchsparse.git@v1.4.0
RUN pip install torch-cluster==1.5.9\
                torch-geometric==1.7.0\
                torch-scatter==2.0.6\
                torch-sparse==0.6.9\
                tqdm==4.60.0\
                torch-geometric==1.7.0\
                torch-scatter==2.0.6\
                torch-sparse==0.6.9\
                tqdm==4.60.0
```
# Conda Envrionment 3dssg
## auto with yml file
```
conda env create --name 3dssg --file environment.yml
```
## mannully build
```
conda create -n 3dssg pytorch=2.0.1 cudatoolkit=10.2
```
or
```
conda create -n 3dssg pytorch=2.0.1 cudatoolkit=10.2 tensorboard trimesh -c conda-forge
pip install onnxruntime
pip install torch-scatter
pip install torch-sparse
pip install torch-cluster
pip install torch-spline-conv
pip install torch-geometric
pip install open3d
```
## activate conda
```
conda activate 3dssg
```
if conda init needed
```
source /opt/conda/etc/profile.d/conda.sh
conda activate 3dssg
```
