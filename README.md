# Docker Desktop Replacement For MacOS

## Background

Docker Desktop will required paid licenses for use starting Jan 31, 2022. While Docker Desktop has some very convenient features that makes launching containers on MacOS seamless, if you are relunctant to continue using it here are instructions using open source software only.

## The components that make up Docker

It is important to understand that Docker only works natively with Linux OS, which means that for it run on Mac (or Windows), there must be something on that OS that mimics Linux. For Windows, that's WSL2, but for Mac there is no builtin feature. This is essentially the value proposition of Docker Desktop as it run a Linux VM under the hood and makes accessing this VM from host MacOS as seamlessly as possible by handling admin tasks like networking, file sharing, etc. between the two environments. It is also important to note that what you are paying in license is this feature, along with a nice GUI, and that other components that are installed with Docker Desktop such as the docker and docker-compose CLI tools are open source applications. Below is a summary table of the components and what will be replaced.

| Component | Docker Desktop | Open Source Alternative
|-----------|----------------|---------------------------
| Linux VM | Paid Component | Hyperkit
| VM manager | Paid Component | Minikube
| containerd | open source | Minikube
| docker cli | open source | docker 
| docker-compose cli | open source | docker-compose 

## Uninstalling Docker Desktop

The easist way to completely uninstall Docker Desktop is to use the uninstall feature that comes with the applications.

## Installation and Setup

### Prerequisites

1. Homebrew is installed
2. Mac is still on the Intel chipset and NOT an M1

If have an M1, the options will be limited to using podman as a complete replacement for docker desktop. Podmac CLI tool works exactly like docker CLI, however it currently does not have a docker-compose equivilent.

### Installing all components

```
# Install hyperkit
brew install hyperkit

# Install minikube
brew install minikube

# Install Docker CLI
brew install docker

# Install Docker-compose
brew install docker-compose
```

### Setup


