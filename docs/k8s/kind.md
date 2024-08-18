---
title: kind
layout: default
parent: k8s
has_children: false
---

### 1. 简介

kind是一个使用 Docker 容器“节点”运行本地 Kubernetes 集群的工具  
kind 主要是为了测试 Kubernetes 本身而设计的，但也可以用于本地开发或CI

### 2. 使用kind创建k8s测试集群

创建一个名为mycluster的三节点集群，指定pod的网段和svc的网段，并且把pki的证书文件挂在到容器外面来

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: mycluster
featureGates:
- GracefulNodeShutdown
nodes:
- role: control-plane
  extraMounts:
  - hostPath: /home/lee/k8s/kind/pki
    containerPath: /etc/kubernetes/pki
- role: worker
  extraMounts:
  - hostPath: /home/lee/k8s/kind/pki
    containerPath: /etc/kubernetes/pki
- role: worker
  extraMounts:
  - hostPath: /home/lee/k8s/kind/pki
    containerPath: /etc/kubernetes/pki
networking:
  podSubnet: "10.244.0.0/16"
  serviceSubnet: "10.96.0.0/12"
```