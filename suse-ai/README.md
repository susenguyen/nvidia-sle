# Cookbooks to deploy SUSE AI

## Add registries.yaml
```
mirrors:
  dp.apps.rancher.io:
    endpoint:
      - "https://dp.apps.rancher.io"
configs:
  "dp.apps.rancher.io":
    auth:
      username: XXX@email.com
      password: XYW==
```

## Helm login
```
helm registry login dp.apps.rancher.io/charts -u XXX@email.com -p XYW==
```

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
global:
  tls:
    source: secret

cert-manager:
  enabled: false

image:
  pullPolicy: IfNotPresent

ollama:
  enabled: false

ollamaUrls:
  - http://ollama:11434

ingress:
  tls: true
  host: "k3s.suse.lab"
  existingSecret: "tls-owui-ingress"

persistence:
  enabled: true
  size: 5Gi
  storageClass: "local-path"
```
```
helm install open-webui --namespace=suse-ai --values=owui-values.yaml oci://dp.apps.rancher.io/charts/open-webui
```
