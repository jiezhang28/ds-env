# Docker Desktop Replacement For MacOS

## Background

Docker Desktop will required paid licenses for use starting Jan 31, 2022. While Docker Desktop has some very convenient features that makes launching containers on MacOS seamless, if you are relunctant to continue using it here are instructions using open source software only.

### The components that make up Docker

It is important to understand that Docker only works natively with Linux OS, which means that for it run on Mac (or Windows), there must be something on that OS that mimics Linux. For Windows, that's WSL2, but for Mac there is no builtin feature. This is essentially the value proposition of Docker Desktop as it runs a Linux VM under the hood and makes accessing this VM from host MacOS as seamlessly as possible by handling admin tasks like networking, file sharing, etc. between the two environments. It is also important to note that other components that are installed with Docker Desktop such as the docker and docker-compose CLI tools are open source applications. Below is a summary table of the components and what will be replaced.

| Component | Docker Desktop | Open Source Alternative
|-----------|----------------|---------------------------
| Linux VM | Paid Component | VirtualBox
| VM manager | Paid Component | Minikube
| containerd | open source | Minikube
| docker cli | open source | docker 
| docker-compose cli | open source | docker-compose 


### VirtualBox vs Hyperkit

Both VirtualBox and Hyperkit are VM hypervisors under the x86 (Intel) architecture. Both can be used with minikube. However, VirtualBox offers a slight advantage when it comes to sharing files between the host system (your mac) and the VM. When used in conjuction with Minikube, VirtualBox will by default mount the host system `/Users` directory into the VM. This will be useful when you in turn create a mount between the VM and the docker container, thereby allowing the docker containers access to the host file system. Hyperkit lacks this integration with minikube but can still offer the same functionality via another method (9P mount), which has performance downsides and is more cumbersome to set up. 

## Uninstalling Docker Desktop

The easiest way to completely uninstall Docker Desktop is to use the uninstall feature that comes with the applications.

## Installation and Setup

### Prerequisites

1. Homebrew is installed
2. Mac is still on the Intel chipset and NOT an M1

As mentioned above, the VM we are using is only supported on the x86 Intel chipset, so if you are using M1 Mac with Apple's ARM chipset, this solution will not work. The only alternative at the time of this writing is podman. Podman CLI tool works exactly like docker CLI, however it currently does not have a docker-compose equivalent.

### Installing all components

```bash
# Install virtualbox
brew install virtualbox

# Install minikube
brew install minikube

# Install Docker CLI
brew install docker

# Install Docker-compose
brew install docker-compose
```

### Setup

```bash
# Configure VM spec
minikube config set cpu 2
minikube config set memory 4000MB

# Start VM
minikube start --no-kubernetes --driver=virtualbox

# Point docker CLI to the location of the containerd service,
# which now resides in the VM that minikube just launched
eval $(minikube docker-env)
```

**Note:** The `--no-kubernetes` flag prevents kubernetes services from starting, which are not needed to run docker containers and will just take up unnecessary resources.

## Examples
Various examples using docker with minikube.

### Running JupyterLab container

Running jupyter lab with volume mount back out to the host Mac where notebook files can be saved

```bash
docker run -it --rm -p 10000:8888 -v "${PWD}":/home/jovyan jupyter/scipy-notebook:b418b67c225b
```

### Running MySQL container

For this example we will run a mysql container that must have the following features
- The host Mac is able to connect to the database
- The data persisted by the mysql container is stored in the host file system and not on a docker volume or inside the VM

```bash
docker run \
-p 3306:3306 \
-v $PWD/data:/var/lib/mysql \
--user 1000:1000 \
-e MYSQL_ROOT_PASSWORD=pass \
mysql --innodb_use_native_aio=0
```

**Explanation of arguments**:
- `-p 3306:3306` - standard port forwarding from VM to container so host mac can connect
- `-v $PWD/data:/var/lib/mysql` - This mounts `/var/lib/mysql` inside the container to access a filepath on the VM. Recall that VM also has a mount back out to the host Mac so this effectively allows the mysql container to save its data on the host Mac filesystem.
- `--user 1000:1000` - mysql docker image is configured with a specific `mysql` linux user and by default will use this user to own all the file it creates in `/var/lib/mysql/`. This exposes a limitation when used in conjunction with the volume mount option above. From the perspective of the VM, the mount to the host file system has a different permission configuration. By default, a VM linux user: `docker` is the owner of these files. To get around this issue, we will force docker to use this user (via the uid number 1000) instead of the `mysql` user inside the container.
- `--innodb_user_native_aio=0` - Another limitation of accessing the host Mac file system from the container. It does not support a specific file system feature, asynchronous I/O, that is used by Mysql by default. Thus, we need to tell mysql to not use this feature via this option.

