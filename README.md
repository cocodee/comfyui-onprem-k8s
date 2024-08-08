# ComfyUI on an On-Premise Kubernetes Cluster

This project aims to provide two different types of ComfyUI services on Kubernetes:
1. API Service for Production
2. Interactive Service for Designers

## Features
- [x] Minikube Cluster for development
- [x] GPU Sharing by Time Slicing
- [x] Ingress Route by [Traefik](https://doc.traefik.io/traefik/)
- [x] [ComfyUI API Service] Cookie-based Session Stickiness with Timeout
- [x] [ComfyUI API Service] Python Test Script for Image Generation
- [ ] [ComfyUI API Service] Horizontal Pod Autoscaling
- [x] [ComfyUI Interactive Service] Authentication and Authorization
- [x] [ComfyUI Interactive Service] Custom Docker Image for JupyterHub SingleUser (+ComfyUI)
- [x] [ComfyUI Interactive Service] ComfyUI Extension for Jupyter Server Proxy 
- [x] [ComfyUI Interactive Service] Profiles for GPU Env / CPU only Env.
- [x] [ComfyUI Interactive Service] Persistent Volume for ComfyUI User Data.
- [x] [ComfyUI Interactive Service] Evicting inactive users.
- [ ] Monitoring (Dashboard for Metric + Logging)

## Prerequisites
- Install [Docker](https://docs.docker.com/engine/install/)
- Install [Kubectl](https://kubernetes.io/docs/tasks/tools/)
- Install [Minikube](https://minikube.sigs.k8s.io/docs/start)
- Login [GHCR](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry#authenticating-with-a-personal-access-token-classic)

## Cluster Setup
```bash
MODEL_PATH=PATH_TO_MODELS make cluster
# Enable GPU Time-Slicing for multiple replicas.
helm install nvidia-device-plugin charts/nvidia-device-plugin -n kube-system
# Install Ingress.
kubectl create namespace ingress
helm install traefik charts/traefik -n ingress

# Volumes.
kubectl apply -f volumes/minikube.yaml  # for minikube env.
```

## ComfyUI API Service
This is an on-premise Kubernetes cluster version inspired by [comfyui on EKS](https://github.com/aws-samples/comfyui-on-eks).

Build ComfyUI (Optional)
```bash
make docker-build
make docker-push
make docker-run  # for testing
```

```bash
helm install comfyui charts/comfyui
```

- Create a connection for testing in minikube: `make tunnel`
- open http://localhost/comfyui
- `cd test`
- `python main.py -s http://host-address/comfyui`

Result:
```bash
Generation started.
Generation not ready, sleep 1s ...
Generation not ready, sleep 1s ...
Generation finished.
Inference finished.
ClientID: 80e564e7-48ef-44cd-aae0-2a18fa091deb.
PromptID: 8882b193-d7d4-42e4-8003-7cdfe51a465d.
Num of images: 1.
Time spent: 2.17s.
------
```

NOTE:
- Set the proper host volume path in `charts/comfyui/values.yaml`

## ComfyUI Interactive Service (w/ JupyterHub)
Build ComfyUI (Optional)
```bash
make docker-build-jupyter
make docker-push-jupyter
make docker-run-jupyter  # for testing
```

```bash
helm install jupyterhub charts/jupyterhub
```

- `python main.py -s http://host-address/hub`
- login with `id: admin / pw: admin123!@#`

<img width="1503" src="https://github.com/user-attachments/assets/251e1c6c-6e46-49c6-9b0a-5f6c58b7d8ef">

## References
### GPU Sharing on K8S
- https://github.com/NVIDIA/k8s-device-plugin?tab=readme-ov-file#with-cuda-time-slicing
- https://nyyang.tistory.com/198

### ComfyUI + K8S
- https://github.com/aws-samples/comfyui-on-eks

### Ingress
- https://doc.traefik.io/traefik/routing/services/?ref=traefik.io#sticky-sessions
- https://doc.traefik.io/traefik/routing/providers/kubernetes-ingress/

### JupyterHub
- https://z2jh.jupyter.org/en/stable/
- https://z2jh.jupyter.org/en/latest/administrator/services.html

### Jupyter Server Proxy
- https://jupyter-server-proxy.readthedocs.io/en/latest/server-process.html
- https://github.com/jupyterhub/jupyter-rsession-proxy/
