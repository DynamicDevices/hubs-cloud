# Hubs Cloud

A fork of the [Hubs Foundation](https://hubsfoundation.org/) Hubs Cloud Community Edition with changes to support installation onto a Raspberry Pi.

The intent is that this repository, and associated repositories, are temporary forks for testing. Ideally when properly validated these repositories will be presented to Hubs Foundation upstream for inclusion in their codebase as a set of pull requests. 

It is understood that these changes could potentially break a lot of behaviours so we're very much looking for people who are interested in testing this out on Raspperry Pis, arm64, and amd64 platform architectures.

# Oustanding Issues

- We've had to disable WebRTC authentication in `coturn` and need to understand why inter-pod database lookups aren't working
- The HTTP LetsEncrypt `certbotbot` process isn't always reliable
- We should have some default media installed for users to get doing

# Installation (Raspberry Pi 5)

We're testing on a Raspberry Pi 5 with 8GB of RAM but this *should* work on a 4GB board and possibly even on a 2GB board variant.

## Setup a 'vanilla' Raspberry Pi OS image

Go through a standard imaging procedure such as using `rpi-imager` [link here](https://www.raspberrypi.com/software) to install a standard **64-bit** Raspberry Pi desktop [image](https://www.raspberrypi.com/software/raspberry-pi-desktop) 

TIP: You can use `rpi-imager` to edit the image settings before writing to the Pi and you might want to put in a default WiFi SSID and password so you can easily connect, and also consider adding a public key for security if you are comfortable with these things. I am going to use the following settings and you may need to pay attention to the user you install as and the hostname in the documentation below

| Setting | Value |
| ------- | ----- |
|Host Name | hubs-pi.local |
|User | hubs |

<img src="https://github.com/user-attachments/assets/12640669-7824-4482-ba27-8260c56747a1" width=50% />

Boot the Raspberry Pi and then login as the `hubs` user

TIP: If you're using the board without a display and keyboard you'll be logging in with the SSH protocol. You'll ethernet connect to your local network with a wired ethernet connection or you may have chosen to set WiFi credentials in the step above. To determine the IP address of the board you can ping the host name in this case `hubs-pi.local`. We use Linux here but I think if memory serves this is `hubs-pi.localdomain` on Windows computers.

```
$ ping hubs-pi.local
PING hubs-pi.local (192.168.0.174) 56(84) bytes of data.
```
Then use SSH or [Putty](https://www.putty.org/) to connect to this hostname or IP address and login

## Option 1: Auaomated Installation

Helper script **TBD**

## Option 2: Manual Installation

### Update installation to latest packages and install needed dependencies

Use the APT package manager to update and install as follows

```
sudo apt update
sudo apt -y upgrade
```

Install needed dependencies

```
sudo apt -y install git build-essential certbot ca-certificates curl
```

### Install Docker

Enter the following to install the Docker container manager. Details are taken from the instructions [here](https://docs.docker.com/engine/install/debian)

```
# Add Docker's official GPG key:
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

```
sudo apt-get -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
The next step is **critical** to that you can run docker containers as the non-root user. Details [here](https://docs.docker.com/engine/install/linux-postinstall/)

```
sudo usermod -aG docker $USER
```

NOTE: You need to log out and log back in for this change to take effect

Finally do a quick test that you can run a docker container

```
docker run hello-world
```
You should see something like this

```
hubs@hubs-pi:~$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c9c5fd25a1bd: Pull complete
Digest: sha256:c41088499908a59aae84b0a49c70e86f4731e588a737f1637e73c8c09d995654
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

The Docker container engine is now installed !!!

### Install MiniKube (Kubernetes)

Now you need to install [MiniKube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Farm64%2Fstable%2Fbinary+download) which is a Kubernetes implementation for 'local' development.

Enter these commands

```
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-arm64
sudo install minikube-linux-arm64 /usr/local/bin/minikube && rm minikube-linux-arm64
```
Check you can run the `minikube` command successfully.

Start your cluster manually. We will automate this with a `systemd` service later on

```
minikube start
```

After a couple of minutes you should see something akin to

```
hubs@hubs-pi:~$ minikube start
* minikube v1.35.0 on Raspbian 12.10 (arm64)
* Automatically selected the docker driver. Other choices: none, ssh
* Using Docker driver with root privileges
* Starting "minikube" primary control-plane node in "minikube" cluster
* Pulling base image v0.0.46 ...
* Downloading Kubernetes v1.32.0 preload ...
    > preloaded-images-k8s-v18-v1...:  314.92 MiB / 314.92 MiB  100.00% 10.95 M
    > gcr.io/k8s-minikube/kicbase...:  452.84 MiB / 452.84 MiB  100.00% 9.10 Mi
* Creating docker container (CPUs=2, Memory=2200MB) ...
* Preparing Kubernetes v1.32.0 on Docker 27.4.1 ...
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Configuring bridge CNI (Container Networking Interface) ...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Enabled addons: storage-provisioner, default-storageclass
* kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

Next install `minikube` `kubectl` support with

```
minikube kubectl -- get po -A
```

We will be using the `kubectl` command extensively to interact with our new kubernetes cluster. The instructions linked to above talk about adding an `alias` to the shell but this doesn't work well with the Hubs scripts. Instead we will use a trick I learnt from (BusyBoxy)[https://busybox.net/] and symlink to the main `minikube` binary as follows

```
sudo ln -s /usr/local/bin/minikube /usr/local/bin/kubectl
```

Run up `kubectl` to test

```
kubectl get pods
```

The Kubernetes container orchestration engine is now installed !!!
