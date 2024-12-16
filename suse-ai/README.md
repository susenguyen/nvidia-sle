# Cookbooks to deploy SUSE AI

## ollama
- ollama-values.yaml (using k3s local-path provisioner for persistent storage)
```
ollama:
  gpu:
    enabled: true

    # For now, only nvidia image is available
    type: 'nvidia'
    number: 1

  # -- List of models to pull at container startup
  models:
    - llama3.2

image:
  pullPolicy: IfNotPresent

persistentVolume:
  enabled: true
  size: 30Gi
  storageClass: "local-path"
```

```
helm install ollama --values=ollama-values.yaml --namespace=suse-ai --create-namespace oci://dp.apps.rancher.io/charts/ollama
```

## open-webui
- Create secret (if using your own ingress TLS certificate)
```
kubectl create secret -n suse-ai tls tls-owui-ingress --cert=server.crt --key=server.key
```
- owui-values.yaml (using k3s local-path provisioner for persistent storage)
```
image:
  pullPolicy: IfNotPresent

ingress:
  tls: true
  host: "slem.suse.lab"
  existingSecret: "tls-owui-ingress"

persistence:
  enabled: true
  size: 5Gi
  storageClass: "local-path"
```
```
helm install open-webui --namespace=suse-ai --values=owui-values.yaml oci://dp.apps.rancher.io/charts/open-webui
```
