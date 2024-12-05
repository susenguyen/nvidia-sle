# SL Micro 6

## Enable SLEM Extras
- transactional-update register -p SL-Micro-Extras/6.0/x86_64

## Deploy C++ dependencies
- gcc
- gcc-build-PIE
- gcc-build
- gcc-build-c++

## transactional-update shell
- zypper ar https://developer.download.nvidia.com/compute/cuda/repos/sles15/x86_64/ nvidia
- Install (Install version 550 (the most recent in this version is ok)
    - nvidia-compute-utils-G06:550.90.12-1
    - nvidia-driver-G06-kmp-default:550.90.12_k4.12.14_150.47-1
    - [only with device-plugin] libnvidia-container1
    - [only with device-plugin] nvidia-container-toolkit

## Configure Runtime
- configure /var/lib/rancher/k3s/agent/etc/containerd/config.toml.tmpl
    - add:
```
```

## Deploy the nvidia-device-plugin
- GPU will be annotated with gpu/nvidia
