# DockContainer_CondaEnv_OnVSCode
INStruction for building container and Conda Environment on VsCode
# Docker Container
1. add a new folder in vscode as the workspace file for docker container
2. add a new folder under the directory with .devcontianer
3. add devcontainer.json file
```
{
    "name": "VLSAT",
    "dockerComposeFile": "docker-compose.yml",
    "runServices": ["devcontainer"],
    "service": "devcontainer",
    "workspaceFolder": "/vlsat",
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
    image: vlsat
    container_name: vlsat
    volumes:
      - /home/wenting/code/VLSAT/.devcontainer:/vlsat/.devcontainer
      - /home/wenting/code/VLSAT/CVPR2023-VLSAT:/vlsat/code/CVPR2023-VLSAT
      - /home/wenting/code/dsg_3d/data/3RScan:/vlsat/data/3RScanPart
      - /home/wenting/code/dsg_3d/data/ScanNetAll:/vlsat/data/ScanNet
      - /home/wenting/code/dsg_3d/data/3RScanAll:/vlsat/data/3RScan
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

RUN apt-get update && apt-get install -y libgl1-mesa-glx

# Create the environment:
COPY environment.yaml ./
RUN conda env create -n vlsat -f environment.yaml
 
SHELL ["/bin/bash", "-c"]
 
# RUN echo "source activate pytorch-gpu" > ~/.bashrc
#ENV PATH /opt/conda/envs/env/bin:$PATH
#ENV PATH /opt/conda/envs

# PATH = [a, b, c, d]
# PATH = [/opt/conda/envs/env/bin, ...PATH]
# PATH = [/opt/conda/envs/env/bin, a, b, c, d]
# PATH = [/opt/conda/envs/env/bin]
 
CMD ["/bin/bash"] 
```
6.command+shift+p
dev container:  open folder in container
choose the file root holds the .devcontainer
The .dev and code file should in seperate folder then build the container under their directory

![Screenshot 2023-09-13 at 15 59 23](https://github.com/WentingXu3o3/DockContainer_CondaEnv_OnVSCode/assets/59476953/03f2e197-4a8b-41d3-9bac-5e23d9a3bacd)


# Conda Envrionment 3dssg
## auto with yml file
```
conda env create --name 3dssg --file environment.yml
```
## mannully build
```
conda create -n 3dssg pytorch=2.0.1 cudatoolkit=10.2
```
```
conda env update -n 3dssg --file environment_new.yml 
```
or python should be 3.10/3.8/3.6 to pip open3d
```
conda create -n 3dssg python=3.10
codna create -n 3dssg python=3.10 pytorch=2.0.1 cudatoolkit=10.2 tensorboard trimesh
and then run py 缺啥pip啥
```
```
conda create -n 3dssg python=3.10 pytorch=2.0.1 cudatoolkit=10.2 tensorboard trimesh -c conda-forge
pip install onnxruntime
pip install torch-scatter
pip install torch-sparse
pip install torch-cluster
pip install torch-spline-conv
pip install torch-geometric
pip install open3d
```
## activate conda
conda env list to see all the conda env
```
conda env list
```
```
conda activate 3dssg
```
if conda init needed
```
source /opt/conda/etc/profile.d/conda.sh
conda activate 3dssg
```
## conda env export to get yaml then we can add to docker file for using the same conda environment next time
```
#when conda activate
conda env export > envrionment.yaml
```

change Dockerfile to
```
FROM pytorch/pytorch:2.0.0-cuda11.7-cudnn8-devel
RUN apt-get update && apt-get install -y \
    git \
    libsparsehash-dev

RUN pip install --upgrade git+https://github.com/mit-han-lab/torchsparse.git@v1.4.0

RUN apt-get update && apt-get install -y libgl1-mesa-glx

# Create the environment:
COPY environment.yaml ./
RUN conda env create -n 3dssg -f environment.yaml
 
SHELL ["/bin/bash", "-c"]
 
# RUN echo "source activate pytorch-gpu" > ~/.bashrc
#ENV PATH /opt/conda/envs/env/bin:$PATH
#ENV PATH /opt/conda/envs

# PATH = [a, b, c, d]
# PATH = [/opt/conda/envs/env/bin, ...PATH]
# PATH = [/opt/conda/envs/env/bin, a, b, c, d]
# PATH = [/opt/conda/envs/env/bin]
 
CMD ["/bin/bash"] 
```
Now docker-compose.yml # save two workspace add more data to the docker.
```
version:  "3.7"
services: 
  devcontainer:
    build: .
    image: 3dssg
    container_name: 3dssg
    volumes:
      - ..:/home/wenting/code/3DSSG
      - /mnt/PRJ-ARIAGlasses:/home/wenting/code/data
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
```
version:  "3.7"
services: 
  devcontainer:
    build: .
    image: 3ddsg
    container_name: 3ddsg
    volumes:
      - /home/wenting/code/dsg_3d/.devcontainer:/dsg_3d/.devcontainer
      - /home/wenting/code/dsg_3d/code:/dsg_3d/code
      - /home/wenting/code/dsg_3d/data/3RScan:/dsg_3d/data/3RScan
      - /home/wenting/code/dsg_3d/data/ScanNet:/dsg_3d/data/ScanNet
      - /mnt/PRJ-ARIAGlasses/3RScan:/dsg_3d/data/3RScanAll
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


devcontainer.json
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
## Docker folder
![Screenshot 2023-07-20 at 22 40 27](https://github.com/WentingXu3o3/DockContainer_CondaEnv_OnVSCode/assets/59476953/51bd7d38-eb68-4ad8-840f-dd84b1ba3405)

## Then can build new container.

## screen
keep the container run something using screen
screen -r
ctrlA ctrlD to back it.
ctrlA : quit to end the screen
# scp file
```
scp -r ubuntu:code/dsg_3d Desktop/python/ 
```
