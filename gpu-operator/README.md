# SL Micro 6

Deployed on RKE2 (k3s doesn't seem to work out of the box with the gpu-operator)

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

## Deploy the nvidia driver & CUDA utilities
- Add the Nvidia Developer repo
```
zypper ar https://developer.download.nvidia.com/compute/cuda/repos/sles15/x86_64/ nvidia
```

- Install (Install version 550 (the most recent in this version is ok)
    - nvidia-compute-utils-G06=550.127.08-1
    - nvidia-driver-G06-kmp-default=550.127.08_k5.14.21_150500.55.83-1
```
zypper in nvidia-compute-utils-G06=550.127.08-1 nvidia-driver-G06-kmp-default=550.127.08_k5.14.21_150500.55.83-1
```
- Reboot the system

## Deploy the gpu-operator
- Add the nvidia helm repository
```
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia && helm repo update
```
- Prepare the values.yaml file
```
toolkit:
  env:
  - name: CONTAINERD_CONFIG
    value: /var/lib/rancher/rke2/agent/etc/containerd/config.toml.tmpl
  - name: CONTAINERD_SOCKET
    value: /run/k3s/containerd/containerd.sock
  - name: CONTAINERD_RUNTIME_CLASS
    value: nvidia
  - name: CONTAINERD_SET_AS_DEFAULT
    value: "true"
```

- Launch the installation
```
helm install gpu-operator -n gpu-operator   --create-namespace nvidia/gpu-operator   --set driver.enabled=false -f values.yaml
```
- Check the validators
```
kubectl logs -n gpu-operator nvidia-operator-validator-kp92t
Defaulted container "nvidia-operator-validator" out of: nvidia-operator-validator, driver-validation (init), toolkit-validation (init), cuda-validation (init), plugin-validation (init)
all validations are successful

kubectl logs -n gpu-operator nvidia-cuda-validator-lk6zz
Defaulted container "nvidia-cuda-validator" out of: nvidia-cuda-validator, cuda-validation (init)
cuda workload validation is successful
```
- Check the device-plugin
```
kubectl logs -n gpu-operator nvidia-device-plugin-daemonset-c7tjf
...
I0114 14:56:40.896708       1 main.go:356] Retrieving plugins.
I0114 14:56:40.921178       1 server.go:195] Starting GRPC server for 'nvidia.com/gpu'
I0114 14:56:40.922338       1 server.go:139] Starting to serve 'nvidia.com/gpu' on /var/lib/kubelet/device-plugins/nvidia-gpu.sock
I0114 14:56:40.925499       1 server.go:146] Registered device plugin for 'nvidia.com/gpu' with Kubelet
```

## Start test pod
- Use the following pod manifest
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
- Check the logs
```
kubectl logs ollama
...
2025/01/14 15:04:44 routes.go:1125: INFO server config env="map[CUDA_VISIBLE_DEVICES: GPU_DEVICE_ORDINAL: HIP_VISIBLE_DEVICES: HSA_OVERRIDE_GFX_VERSION: OLLAMA_DEBUG:false OLLAMA_FLASH_ATTENTION:false OLLAMA_HOST:http://0.0.0.0:11434 OLLAMA_INTEL_GPU:false OLLAMA_KEEP_ALIVE:5m0s OLLAMA_LLM_LIBRARY: OLLAMA_MAX_LOADED_MODELS:0 OLLAMA_MAX_QUEUE:512 OLLAMA_MODELS:/root/.ollama/models OLLAMA_NOHISTORY:false OLLAMA_NOPRUNE:false OLLAMA_NUM_PARALLEL:0 OLLAMA_ORIGINS:[http://localhost https://localhost http://localhost:* https://localhost:* http://127.0.0.1 https://127.0.0.1 http://127.0.0.1:* https://127.0.0.1:* http://0.0.0.0 https://0.0.0.0 http://0.0.0.0:* https://0.0.0.0:* app://* file://* tauri://*] OLLAMA_RUNNERS_DIR: OLLAMA_SCHED_SPREAD:false OLLAMA_TMPDIR: ROCR_VISIBLE_DEVICES:]"
time=2025-01-14T15:04:44.709Z level=INFO source=images.go:782 msg="total blobs: 0"
time=2025-01-14T15:04:44.709Z level=INFO source=images.go:790 msg="total unused blobs removed: 0"
time=2025-01-14T15:04:44.709Z level=INFO source=routes.go:1172 msg="Listening on [::]:11434 (version 0.3.6)"
time=2025-01-14T15:04:44.710Z level=INFO source=payload.go:30 msg="extracting embedded files" dir=/tmp/ollama3902969706/runners
time=2025-01-14T15:04:51.096Z level=INFO source=payload.go:44 msg="Dynamic LLM libraries [cpu cpu_avx cpu_avx2 cuda_v11 rocm_v60102]"
time=2025-01-14T15:04:51.096Z level=INFO source=gpu.go:204 msg="looking for compatible GPUs"
time=2025-01-14T15:04:51.516Z level=INFO source=types.go:105 msg="inference compute" id=GPU-e8fabfc1-3ab2-829d-f4a9-18d2d83e8259 library=cuda compute=6.1 driver=12.4 name="NVIDIA GeForce MX150" total="1.9 GiB" available="1.9 GiB"
```
