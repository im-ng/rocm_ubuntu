# rocm_ubuntu

In this setup guide I am targeting the installation of `ROCm 3.7` for `RX480 GPU`. But the steps mostly suits for latest GPU's as well.

Please refer specific version [release notes](https://github.com/RadeonOpenCompute/ROCm/tree/roc-3.7.x) for linux kernel and GPU architecture support. *I ended up wasting more time before reading this properly*.

Prefer to get start with **fresh** `Ubuntu` 20.04 Installation


### Downgrade to 5.6 oem Kernel
```
❯ sudo apt update
❯ sudo apt upgrade
❯ sudo apt install linux-image-5.6.0-1042-oem linux-headers-5.6.0-1042-oem
❯ sudo apt remove linux-image-5.8.0-43-generic linux-headers-5.8.0-43-generic
❯ sudo apt autoremove
❯ sudo apt-mark hold linux-image-generic linux-headers-generic
❯ sudo rm -rv ${TMPDIR:-/var/tmp}/mkinitramfs-*
❯ dpkg -l | tail -n +6 | grep -E 'linux-image-[0-9]+'
❯ sudo update-initramfs -d -k 5.8.0-41-generic
❯ cd /boot/
❯ sudo rm vmlinuz-5.8.0-43-generic System.map-5.8.0-43-generic config-5.8.0-43-generic
❯ sudo update-grub
❯ sudo reboot
```
### Verify 5.6 Kernel Version
```
❯ uname -a
Linux ryzen 5.6.0-1042-oem #46-Ubuntu SMP Thu Jan 7 00:51:04 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
```
### Install Specific ROCm 
```
❯ sudo apt install libnuma-dev
❯ wget -q -O - https://repo.radeon.com/rocm/rocm.gpg.key | sudo apt-key add -
❯ echo 'deb [arch=amd64] https://repo.radeon.com/rocm/apt/3.7/ xenial main' | sudo tee /etc/apt/sources.list.d/rocm.list
❯ sudo apt update
❯ sudo apt install rocm-dkms
❯ sudo usermod -a -G video $LOGNAME
❯ sudo usermod -a -G render $LOGNAME
❯ sudo echo 'export PATH=$PATH:/opt/rocm/bin:/opt/rocm/rocprofiler/bin:/opt/rocm/❯ opencl/bin' | sudo tee -a /etc/profile.d/rocm.sh
❯ sudo reboot
```

* Look at the debian source list being added.

### Check ROCm Installation
```
❯ rocm-smi
========================ROCm System Management Interface========================
================================================================================
GPU  Temp   AvgPwr  SCLK    MCLK    Fan     Perf  PwrCap  VRAM%  GPU%  
0    43.0c  7.163W  608Mhz  300Mhz  13.73%  auto  145.0W   11%   0%    
================================================================================
==============================End of ROCm SMI Log ==============================
❯ rocminfo | grep Marketing
  Marketing Name:          AMD Ryzen 5 2600 Six-Core Processor
  Marketing Name:          Ellesmere [Radeon RX 470/480/570/570X/580/580X/590]

❯ clinfo | grep OpenCL
  Platform Version:				 OpenCL 2.0 AMD-APP (3182.0)
    Execute OpenCL kernels:			 Yes
  Device OpenCL C version:			 OpenCL C 2.0 
  Version:					 OpenCL 1.2 

❯ sudo apt install rocm-bandwidth-test
❯ rocm-bandwidth-test 
❯ sudo apt show -a rocm-dev | grep Version

Version: 3.7.0-20
❯ 
```
### Install Docker 

```
❯ sudo apt remove docker docker-engine docker.io containerd runc
❯ sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
❯ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
❯ sudo apt-key fingerprint 0EBFCD88
❯ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
❯ sudo apt update
❯ sudo apt-get install docker-ce docker-ce-cli containerd.io
❯ sudo groupadd docker
❯ sudo usermod -a -G docker $LOGNAME
❯ sudo docker images
❯ sudo docker ps
❯ sudo reboot
❯ docker images
❯ docker run hello-world
```

### Run PyTorch on Docker Container
- It takes quite some time to download image.
    ```
    ❯ docker pull rocm/pytorch:rocm3.7_ubuntu18.04_py3.6_pytorch
    ```
- Run as docker container
    ```
    ❯ docker run -it -v $HOME:/data --privileged --rm --device=/dev/kfd --device=/dev/dri --group-add video rocm/pytorch:rocm3.7_ubuntu18.04_py3.6_pytorch
    ```

* Verify Cuda Status
    ```
    ❯ docker run -it -v $HOME:/home --privileged --rm --device=/dev/kfd --device=/dev/dri --group-add video rocm3.7_pytorch1.7:latest
    root@2f3ba661752e:~/pytorch# cd ~
    root@2f3ba661752e:~# pwd
    /root
    root@2f3ba661752e:~# python
    Python 3.6.9 (default, Jul 17 2020, 12:50:27) 
    [GCC 8.4.0] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import torch
    >>> torch.cuda.is_available()
    True
    >>> torch.cuda.get_device_name(0)
    'Device 67df'
    >>> exit()
    root@2f3ba661752e:~#
    ```


### References
1. https://github.com/RadeonOpenCompute/ROCm/tree/roc-3.7.x
2. https://stackoverflow.com/questions/65987212/failed-to-install-rocm-on-ubuntu-20-04