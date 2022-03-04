# Docker Desktop Replacement For MacOS

## Background

Docker Desktop will required paid licenses for use starting Jan 31, 2022. While Docker Desktop has some very convenient features that makes launching containers on MacOS seamless, if you are relunctant to continue using it here are instructions using open source software only.

## The components that make up Docker

It is important to understand that Docker only works natively with Linux OS, which means that for it run on Mac (or Windows), there must be something on that OS that mimics Linux. For Windows, that's WSL2, but for Mac there is no builtin feature. This is essentially the value proposition of Docker Desktop as it runs a Linux VM under the hood and makes accessing this VM from host MacOS as seamlessly as possible by handling admin tasks like networking, file sharing, etc. between the two environments. It is also important to note that what you are paying in the license is this feature, along with a nice GUI, and that other components that are installed with Docker Desktop such as the docker and docker-compose CLI tools are open source applications. Below is a summary table of the components and what will be replaced.

| Component | Docker Desktop | Open Source Alternative
|-----------|----------------|---------------------------
| Linux VM | Paid Component | VirtualBox
| VM manager | Paid Component | Minikube
| containerd | open source | Minikube
| docker cli | open source | docker 
| docker-compose cli | open source | docker-compose 


## VirtualBox vs Hyperkit

Both VirtualBox and Hyperkit are VM hypervisors under the x86 (Intel) architecture. Both can be used with minikube. However, VirtualBox offers a slight advantage when it comes to sharing files between the host system (your mac) and the VM. When used in conjuction with Minikube, VirtualBox will by default mount the host system `/Users` directory into the VM. This will be useful when you in turn create a mount between the VM and the docker container, thereby allowing the docker containers access to the host file system. Hyperkit lacks this integration with minikube but can still offer the same functionality via another method (9P mount), which has performance downsides and is more cumbersome to set up. 

## Uninstalling Docker Desktop

The easiest way to completely uninstall Docker Desktop is to use the uninstall feature that comes with the applications.

## Installation and Setup

### Prerequisites

1. Homebrew is installed
2. Mac is still on the Intel chipset and NOT an M1

As mentioned above, the VM we are using is only supported on the x86 Intel chipset, so if you are using M1 Mac with Apple's ARM chipset, this solution will not work. The only alternative at the time of this writing is podman. Podman CLI tool works exactly like docker CLI, however it currently does not have a docker-compose equivilent.

### Installing all components

```
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

```
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

### Running MySQL container

For this example we will run a mysql container that must have the following features
- The host Mac is able to connect to the database
- The data persisted by the mysql container is stored in the host file system and not on a docker volume or inside the VM

```bash
docker run -v $PWD/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=pass \
-p 3306:3306 \
--user 1000:1000 \
mysql --innodb_use_native_aio=0
```