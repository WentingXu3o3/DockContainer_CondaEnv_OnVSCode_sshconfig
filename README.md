
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
images name only could be in lowercase! 
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
--What is the benefit if you build a conda environment in your docker container?

--Firstly, if the docker container has not been delete, the conda would always there.

--Secondly, if you want to build different test environment in the docker container, conda would help with

--Thirdly, docker seems like still would change the python libraries, while conda would not.

!! make sure you have activated the conda environment, with the root showing conda env name in the first. and the running root would be /opt/conda/"condaenvname"/something  /dockerworkspace/something/main.py
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
source where is the conda
```
source /opt/conda/etc/profile.d/conda.sh
conda activate 3dssg
```
or
```
conda init
```
then kill the current terminal and get a new one to activate the conda env
!! pay attention to the source of the debugging interpreter on the right down, it should change to opt/conda
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
#activate gpu in docker container

run on host to see
```
docker run --gpus all 3dssg:latest nvidia-smi
```
or try with a light weight image (find a proper cuda version not higher than the cuda version in the host by nvidia-smi)
```
docker run --rm --gpus all nvidia/cuda:12.2.0-runtime-ubuntu20.04 nvidia-smi
```
## Docker folder
![Screenshot 2023-07-20 at 22 40 27](https://github.com/WentingXu3o3/DockContainer_CondaEnv_OnVSCode/assets/59476953/51bd7d38-eb68-4ad8-840f-dd84b1ba3405)

## Then can build new container.

## screen
keep the container run something using screen
1. build a new screen
```
screen
```
2. in the new screen to quit and deattach but keep the screen
```
ctrl a+d
```
3. to see the screens already existing
```
screen -ls
```
4. to reattach one screen
```
screen -r <screen-id>
```
5. quit but not end is same with step2
6. quit and end the screen
```
ctrl a
```
7.kill the screen in terminal
```
kill <screen-id>
```
# scp file 拷贝远程文件到本地
for recursive all file copied
```
scp -r ubuntu:code/dsg_3d Desktop/python/ 
```
for individual file
```
scp directory/becopied directory/topaste
```
for skip existing files:
```
rsync -avz --progress -e "ssh -p 22141" /dsg_3d/data/Matterport3D hostname@ip:~/WentingCode/code/dsg_3d/data/
```
```
rsync -avz --progress --ignore-existing -e ssh ubuntu:code/dsg_3d/data/3RScanAll/ ./data/3RScanAll/
```
# ssh config
ssh config 文件
好处：自定义sever名称，不用每次输ip
私钥和公钥：可以在本地保存一个私钥（可以不用输passphrase），生成的公钥同时保存在不同的sever上，然后再config文件中添加identifyfile从而每次自动用本地的私钥验证。远离每次都要输account密码

1. build a private key and paired public key
```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/my_private_key -N ""
```
-N means no passphrase for this private key but also you can save a passphrase

you can check with 
```
ls ~/.ssh/
```
and see my_private_key and my_private_key.pub

2. enter config file to save the host name and ip address and user name with port
```
nano ~/.ssh/config
```
IdentityFile: where to save your private key 
and you can rename your host

``` in config file
Host ubuntu
  HostName 10.70.xxx.xxx
  User wenting
  IdentityFile ~/.ssh/my_private_key

Host abcd
  HostName 
  User axxxx
  Port xxxxx
  IdentityFile ~/.ssh/my_private_key
```




3. then copy public key to the sever's authorized_keys:
```
scp ~/.ssh/my_private_key.pub your_username@your_server_ip:~/.ssh/
scp ~/.ssh/my_private_key.pub ubuntu:~/.ssh/
```
if the server with port then will be:
```
scp -P XXXX ~/.ssh/my_private_key.pub your_username@your_server_ip:~/.ssh/
```
 (if already save the port and host name can be changed to)
```
scp ~/.ssh/my_private_key.pub abcd:~/.ssh/
scp -r ~/ abcd:~/ means all files in folder copy
```

4. in sever terminal cat the public key to authorized_keys
```
cat ~/.ssh/my_private_key.pub >> ~/.ssh/authorized_keys
```


5. then done! you can ssh the sever without password and using a customized name!
```
ssh ubuntu
```
also you can delete the public key on server after done.
```
rm ~/.ssh/my_private_key.pub
```
attention: if not work, check all .ssh and key file in this access mode.
```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
# gui from ubuntu to mac
1. Download Xquartz on Mac
2. in mac termianl export DISPLAY=:0
3. try with xeyes in terminal to see the display
4. in mac termianl start ssh -X ip
5. if doesn't work, in mac nano ~/.ssh/config
   add ForwardX11 yes
Host ubuntu
  HostName 10.70.152.164
  User wenting
  IdentityFile ~/.ssh/my_private_key
  ForwardX11 yes
4.try with xeyes on ssh

# Remote error with VSCode
Failed to connect to the remote extension host server (Error: WrappedError(WrappedError { message: "error creating temp download dir", original: "No space left on device (os error 28) at path \"/tmp/.tmpVfSb1P\"" }))

1. check the space:
disk usage -human readable
```
du -h
```
and if u want to see particular folder
```
du -h --max-depth=1 /
```
to read all folder in the root "/" and only shows the non null
```
du -h --max-depth=1 / 2>dev/null
```

```
du -h --max-depth=1 /var
```
# unzip all zip files in the sub directories of current directory

```
find . -name "*.zip" -exec sh -c 'unzip -n -d "$(dirname "$1")" "$1"' _ {} \;
find ./tasks -name "*.zip" -exec sh -c 'mkdir -p ./task_unzip/"$(dirname "${1#./tasks/}")" && unzip -n -d ./task_unzip/"$(dirname "${1#./tasks/}")" "$1"' _ {} \;
```
# File Management
1. Reports total disk space usage in a human-readable format
```
df -h
```
2. Reports the disk usage of a directory and all its contents
```
du -sh
du -sh ./
```
3. lsblk provides a tree-like structure to show the disk layout and usage
4. ncdu .   a great interactive tool to dynamically check disk usage in the current directory and explore subdirectories.
