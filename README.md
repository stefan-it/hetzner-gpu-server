# Hetzner GPU Server Setup

This repository hosts my cheatsheet for Hetzner GPU Server Setup. Hopefully, I could also be useful for others ;)

I opened the [discussion section](https://github.com/stefan-it/hetzner-gpu-server/discussions) so that other GEX44 users can discuss various things.

## GEX44

In March 2024, Hetzner thanksfully released a new [generation of GPU servers](https://www.hetzner.com/press-release/new-gpu-server/), after their [first GPU server](https://web.archive.org/web/20210613172423/https://www.hetzner.com/news/blitzschnell-gestochen-scharf-neuer-dedicated-root-server-ex51-ssd-gpu/) back in March 2017!

The GEX44 comes with an Intel® Core™ i5-13500, 64GB of RAM and a NVIDIA RTX™ 4000 with 20GB GPU-RAM.

I will use the GPU mainly for fine-tuning NLP models with the awesome [Flair](https://github.com/flairNLP/flair) library. I am very excited about the new GEX44 server, as I've also used the old GPU server back in 2018 for releasing a lot of [Flair Embeddings](https://github.com/flairNLP/flair-lms)!


# Initial setup

After using Ubuntu 22.04 (installed from rescue console) the following steps are done in the initial setup stage:

* Updating and upgrading all Ubuntu 22.04 packages
* Upgrade from Ubuntu 22.04 to 23.10
* Install CUDA and NVIDIA drivers

On commandline it looks like:

```bash
$ apt update && apt upgrade -y
$ reboot # To make sure ;)
$ do-release-upgrade # It is not working, because 23.10 is no LTS, so... set "PROMPT" to "normal" in:
$ vim /etc/update-manager/release-upgrades
$ do-release-upgrade
```

Now we have a working Ubuntu 23.10 installation. Now all NVIDIA-related stuff is installed:

```bash
$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
$ dpkg -i cuda-keyring_1.1-1_all.deb
$ apt-get update
$ apt-get -y install cuda-toolkit-12-4
```

Ah! It is not working, due to missing `libtinfo5`, so we download and install it manually:

```bash
$ wget http://security.ubuntu.com/ubuntu/pool/universe/n/ncurses/libtinfo5_6.4-2ubuntu0.1_amd64.deb
$ dpkg -i libtinfo5_6.4-2ubuntu0.1_amd64.deb
$ sudo apt-get -y install cuda-toolkit-12-4
```

After that we install NVIDIA drivers and reboot our nice GEX44:

```bash
$ apt-get install -y cuda-drivers
$ reboot
```

Now let's see if it was working:

```bash
$ nvidia-smi
Wed Mar  6 11:59:03 2024       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.54.14              Driver Version: 550.54.14      CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA RTX 4000 SFF Ada ...    Off |   00000000:01:00.0 Off |                  Off |
| 30%   31C    P8              5W /   70W |       1MiB /  20475MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
                                                                                         
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```

Amazing! The GEX44 is now ready to fine-tune nice models 🎉

# User account

We create a new (unpriviliged) user and add it to the `sudoers` list:

```bash
$ useradd -m -g users -s /bin/bash stefan
$ passwd stefan # Set a nice password here
$ usermod -aG sudo stefan
```

Then login with the newly created user.

# PyTorch installation

PyTorch is installed in a fresh virtual environment (via `venv`), as this is the easiest way to get it running.
Possible alternatives are e.g. Anaconda or using Docker with NVIDIA support and e.g. an [NVIDIA PyTorch image](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/pytorch)).

The following steps will install a perfectly working PyTorch installation with GPU support:

```bash
$ sudo apt install python3-venv
$ python3 -m venv venvs/dev
$ source venvs/dev/bin/activate
```

Now let's test if CUDA is available via:

```bash
$ python3
Python 3.11.6 (main, Oct  8 2023, 05:06:43) [GCC 13.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import torch
>>> torch.cuda.is_available()
True
>>>
```

Now we can start installing more libraries and fine-tuning own models!
