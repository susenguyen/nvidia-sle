# SL Micro 6

## Enable SLEM Extras
- Open a transactional-update shell and then

```
SUSEConnect -p SL-Micro-Extras/6.0/x86_64
```

## Deploy C++ dependencies
- gcc
- gcc-build-PIE
- gcc-build
- gcc-build-c++

```
zypper in gcc gcc-build-PIE gcc-build gcc-build-c++
```

## Deploy the nvidia driver, CUDA utilities and k8s conntainer toolkit
- Add the Nvidia Developer repo
```
zypper ar https://developer.download.nvidia.com/compute/cuda/repos/sles15/x86_64/ nvidia
```

- Install (Install version 550 (the most recent in this version is ok)
    - nvidia-compute-utils-G06:550.90.12-1
    - nvidia-driver-G06-kmp-default:550.90.12_k4.12.14_150.47-1
    - [only with device-plugin] libnvidia-container1
    - [only with device-plugin] nvidia-container-toolkit
```
zypper in nvidia-compute-utils-G06=550.90.12-1 nvidia-driver-G06-kmp-default=550.90.12_k4.12.14_150.47-1 libnvidia-container1 nvidia-container-toolkit
```

## Configure Runtime
- configure /var/lib/rancher/k3s/agent/etc/containerd/config.toml.tmpl
    - add:
```
[plugins."io.containerd.grpc.v1.cri".containerd]
  ...
  default_runtime_name = "nvidia" 
...

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes."nvidia"]
  privileged_without_host_devices = false
  runtime_engine = ""
  runtime_root = ""
  runtime_type = "io.containerd.runc.v2"

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes."nvidia".options]
  BinaryName = "/usr/bin/nvidia-container-runtime"
  SystemdCgroup = true
```

## Deploy the nvidia-device-plugin
- Deploy the device plugin daemonset
```
kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/refs/heads/main/deployments/static/nvidia-device-plugin.yml
```
- GPU will be annotated with gpu/nvidia
```
Capacity:
  ...
  nvidia.com/gpu:     1
  pods:               110
```

## Start test pod
```
apiVersion: v1
kind: Pod
metadata:
  name: ollama
spec:
  containers:
    - name: ollama
      image: ollama/ollama:0.3.6
      imagePullPolicy: IfNotPresent
      resources:
        limits:
          nvidia.com/gpu: 1
```
