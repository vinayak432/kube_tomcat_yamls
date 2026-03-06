<div align="center">

# ☸️ Kubernetes on AWS — Complete DevOps Lab

<img src="https://img.shields.io/badge/Kubernetes-1.33-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white"/>
<img src="https://img.shields.io/badge/AWS-EC2%20%7C%20EBS%20%7C%20EFS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white"/>
<img src="https://img.shields.io/badge/Runtime-Containerd-5C4EE5?style=for-the-badge&logo=containerd&logoColor=white"/>
<img src="https://img.shields.io/badge/CNI-Calico-F58113?style=for-the-badge"/>
<img src="https://img.shields.io/badge/Setup-kubeadm-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white"/>

<br/>

> *"Real Kubernetes skills aren't built in playgrounds — they're forged on actual cloud infrastructure."*

**A battle-ready, real-world Kubernetes lab running on AWS EC2 with kubeadm.**  
Jenkins. Nginx. Tomcat. EBS. EFS. StatefulSets. PostgreSQL. Resource Management. The full DevOps stack — in raw YAML.

[🚀 Quick Start](#-quick-start) • [📂 Structure](#-repository-structure) • [💾 Storage Labs](#-storage-labs) • [🎯 Scheduling](#-scheduling-strategies) • [🧪 Lab Guide](#-step-by-step-lab-guide)

</div>

---

## 🗺️ What's Inside This Lab?

This isn't a toy cluster. This repo simulates **production-grade Kubernetes patterns** on real AWS infrastructure. Every YAML file teaches a concept that matters in real DevOps jobs.

| 🧩 Category | What You'll Practice |
|------------|----------------------|
| 🚀 **Workloads** | Pod → ReplicaSet → Deployment → DaemonSet → StatefulSet |
| 💾 **Storage** | HostPath, Static PV, Dynamic EBS (RWO), Dynamic EFS (RWX) |
| 🔧 **Configuration** | ConfigMaps, Secrets, imagePullSecrets, PostgreSQL Secrets |
| 🌐 **Networking** | ClusterIP, NodePort, AWS LoadBalancer |
| 🎯 **Scheduling** | NodeSelector, NodeAffinity (hard/soft), PodAffinity, PodAntiAffinity, Tolerations |
| 📊 **Resource Management** | ResourceQuota, LimitRange, Guaranteed QoS pods |
| ☁️ **AWS Integration** | EBS CSI Driver, EFS CSI Driver, AWS CCM |
| 🧑‍💻 **Apps** | Jenkins, Nginx, Tomcat, PostgreSQL |

---

## 🏗️ Architecture Overview

```
                        ┌──────────────────────────────────────┐
                        │             AWS Cloud                │
                        │                                      │
                        │   ┌──────────────────────────────┐   │
                        │   │            VPC               │   │
                        │   │                              │   │
                        │   │  ┌─────────────┐  ┌──────────────────┐  │
                        │   │  │Control Plane│  │  Worker Nodes    │  │
                        │   │  │  kubeadm    │  │  EC2 Instances   │  │
                        │   │  │  API Server │  │  kubelet         │  │
                        │   │  │  Scheduler  │  │  containerd      │  │
                        │   │  │  etcd       │  │  Calico CNI      │  │
                        │   │  └─────────────┘  └──────────────────┘  │
                        │   │                          │          │
                        │   │           ┌──────────────┼──────┐   │
                        │   │           │              │      │   │
                        │   │       HostPath        EBS CSI  EFS CSI
                        │   │      (local dev)      (RWO)    (RWX) │
                        │   └──────────────────────────────────┘   │
                        └──────────────────────────────────────────┘
                                             │
                              Jenkins | Nginx | Tomcat | PostgreSQL
```

---

## 📂 Repository Structure

```
k8s_yamls/
│
├── 🧑‍💻 jenkins/                        # Jenkins workload examples
│   ├── jenkins-pod.yaml               # Bare pod (learn the basics)
│   ├── jenkins-deployment.yaml        # Production-style deployment
│   ├── jenkins-daemonset.yaml         # Run on every node
│   └── jenkins-service.yaml           # Expose via LoadBalancer
│
├── 🌐 nginx/                          # Rich scheduling examples
│   ├── nginx-pod.yaml
│   ├── nginx-rs.yaml
│   ├── nginx-deployment.yaml
│   ├── nginx-daemonset.yaml
│   ├── nginx-service_lb.yaml
│   ├── nginx_deploy_nodeselector.yaml      # Schedule to specific node
│   ├── nginx_deploy_nodeaff.yaml           # Node affinity rules
│   ├── nginx_deploy_podaff.yaml            # Co-locate pods
│   └── nginx_deploy_podantiaff.yaml        # Spread pods across nodes
│
├── ☕ tomcat/
│   ├── tomcat-pod.yaml
│   ├── tomcat-deployment.yaml
│   ├── tomcat-daemonset.yaml
│   └── tomcat-service_lb.yaml
│
└── 📦 volumes/                        # All storage & config resources
    │
    ├── 🔧 config_secrets/             # ConfigMaps & general secrets
    │   ├── config_map.yaml
    │   ├── config_deployment_pod.yaml
    │   ├── economic-app-deploy.yaml
    │   └── secret-pod-registry.yaml
    │
    ├── 🔐 secrets/postgres-secret/    # PostgreSQL secret patterns ⭐ NEW
    │   ├── postgres-secret-env-pod.yaml    # Inject secret as env vars
    │   ├── postgres-secret-vol-pod.yaml    # Mount secret as volume
    │   ├── statefulset.yaml               # PostgreSQL StatefulSet
    │   ├── guaranteed-pod.yaml            # Guaranteed QoS class pod
    │   ├── limitrange.yaml               # Namespace-level default limits
    │   ├── resourcequota.yaml            # Namespace resource cap
    │   └── nginx-dev-pod.yaml            # Dev pod with resource limits
    │
    ├── 🖥️ hostpath/                   # Local storage (dev only)
    │   ├── pv.yaml
    │   ├── pvc.yaml
    │   ├── pvc_mnt.yaml
    │   └── jen_host.yaml
    │
    ├── 💿 dynamic-ebs/                # AWS Block Storage (RWO)
    │   ├── ebs-sc.yaml
    │   ├── ebs-test-pvc.yaml
    │   └── ebs-test-pod.yaml
    │
    ├── 📂 dynamic-efs/                # AWS Shared Storage (RWX)
    │   ├── efs-sc.yaml
    │   ├── efs-test-pvc.yaml
    │   ├── efs-test-pod.yaml
    │   ├── efs-test-pod-2.yaml        # Two pods sharing same volume!
    │   ├── jenkins-efs-test.yaml
    │   └── jen-test-pod-antiaffy.yaml
    │
    ├── 💿 static_ebs/
    │   └── ebs-static.yaml            # All-in-one [PV + PVC + Pod]
    │
    └── 📂 static_efs/
        └── efs-static.yaml            # All-in-one [PV + PVC + Pod]
```

---

## 🚀 Quick Start

**Prerequisites:**
- AWS account with EC2 instances
- `kubectl` configured against your cluster
- AWS EBS & EFS CSI drivers installed (see [Phase 2](#-phase-2--install-aws-csi-drivers))

```bash
# Clone the repo
git clone https://github.com/vinayak432/k8s_yamls.git
cd k8s_yamls

# Verify your cluster is ready
kubectl get nodes
kubectl get pods -n kube-system

# Deploy Jenkins (the classic starting point)
kubectl apply -f jenkins/jenkins-deployment.yaml
kubectl apply -f jenkins/jenkins-service.yaml

# Watch it come alive
kubectl get pods -w
```

---

## 💾 Storage Labs

Storage is where most Kubernetes learners get stuck. This repo covers **every storage pattern** you'll encounter in production.

### The Storage Spectrum

```
Dev ◄─────────────────────────────────────────────────────► Production
     HostPath      Static PV       EBS Dynamic      EFS Dynamic
    (node disk)   (pre-created)    (auto block)   (shared, multi-pod)
        │               │               │                │
     Single           Single         Single           Multiple
      Pod              Pod             Pod              Pods
      RWO              RWO             RWO              RWX
```

---

### 🖥️ HostPath — Local Dev Storage

```bash
kubectl apply -f volumes/hostpath/pv.yaml
kubectl apply -f volumes/hostpath/pvc.yaml
kubectl apply -f volumes/hostpath/pvc_mnt.yaml
```
> ⚠️ **Dev only.** Data lives on the node — if the pod moves, data stays behind.

---

### 💿 Dynamic AWS EBS — Block Storage (RWO)

Best for: databases, single-writer workloads

```bash
kubectl apply -f volumes/dynamic-ebs/ebs-sc.yaml       # Create StorageClass
kubectl apply -f volumes/dynamic-ebs/ebs-test-pvc.yaml  # Request storage
kubectl apply -f volumes/dynamic-ebs/ebs-test-pod.yaml  # Mount and test
```

```
StorageClass → PVC → EBS CSI Driver → AWS EBS Volume → Pod
```

> ✅ EBS auto-provisions a volume when the PVC is created. Magic.

---

### 📂 Dynamic AWS EFS — Shared Storage (RWX)

Best for: Jenkins workspaces, shared config, multi-pod writes

```bash
kubectl apply -f volumes/dynamic-efs/efs-sc.yaml
kubectl apply -f volumes/dynamic-efs/efs-test-pvc.yaml
kubectl apply -f volumes/dynamic-efs/efs-test-pod.yaml
kubectl apply -f volumes/dynamic-efs/efs-test-pod-2.yaml  # Second pod, same volume!
```

```
Pod A ──┐
        ├──► PVC ──► EFS CSI ──► AWS EFS Filesystem
Pod B ──┘
```
> ✅ Both pods read/write the **same** EFS filesystem simultaneously. Try it!

---

### ⚡ EBS vs EFS — Quick Reference

| Feature | EBS | EFS |
|---------|-----|-----|
| Access Mode | `ReadWriteOnce` | `ReadWriteMany` |
| Storage Type | Block | File (NFS) |
| Multi-Pod? | ❌ One node only | ✅ Many pods |
| Multi-AZ | ❌ AZ-bound | ✅ Spans AZs |
| Use Case | Databases | Jenkins, shared files |
| CSI Driver | `ebs.csi.aws.com` | `efs.csi.aws.com` |

---

## 🔐 Secrets & PostgreSQL Patterns

The `volumes/secrets/postgres-secret/` folder demonstrates real-world secret management for databases.

### Two Ways to Consume Secrets

```bash
# Method 1: Secret as Environment Variables
kubectl apply -f volumes/secrets/postgres-secret/postgres-secret-env-pod.yaml

# Method 2: Secret as Mounted Volume (file-based)
kubectl apply -f volumes/secrets/postgres-secret/postgres-secret-vol-pod.yaml
```

| Method | How It Works | Best For |
|--------|-------------|----------|
| **Env Vars** | Secret key → `$POSTGRES_PASSWORD` in container | Simple apps |
| **Volume Mount** | Secret key → file at `/etc/secrets/password` | Apps that read config files |

---

### 🗄️ PostgreSQL StatefulSet

```bash
kubectl apply -f volumes/secrets/postgres-secret/statefulset.yaml
```

StatefulSets give PostgreSQL pods:
- **Stable pod names** — `postgres-0`, `postgres-1` (predictable, not random)
- **Individual PVCs** — each pod gets its own persistent volume
- **Ordered startup** — `postgres-0` starts before `postgres-1`
- **Sticky identity** — pod restarts keep the same name and storage

---

## 📊 Resource Management

The `volumes/secrets/postgres-secret/` folder also covers Kubernetes resource governance — critical for multi-team clusters.

### ResourceQuota — Namespace-Level Caps

Limits **total** resources consumable in a namespace:

```bash
kubectl apply -f volumes/secrets/postgres-secret/resourcequota.yaml
kubectl describe resourcequota -n <namespace>
```

```yaml
# Example: Team can't consume more than this in total
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"
```

---

### LimitRange — Per-Pod Default Limits

Sets **default and max limits** for individual pods/containers in a namespace:

```bash
kubectl apply -f volumes/secrets/postgres-secret/limitrange.yaml
```

```yaml
# Any pod without explicit limits gets these defaults applied
spec:
  limits:
  - type: Container
    default:
      cpu: "500m"
      memory: "256Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    max:
      cpu: "2"
      memory: "2Gi"
```

---

### Guaranteed QoS — Priority Pod

```bash
kubectl apply -f volumes/secrets/postgres-secret/guaranteed-pod.yaml
```

Kubernetes assigns **QoS classes** to pods based on resource config:

| QoS Class | When | Eviction Priority |
|-----------|------|------------------|
| **Guaranteed** | `requests == limits` for all containers | Last to be evicted |
| **Burstable** | `requests < limits` or partial | Middle |
| **BestEffort** | No requests or limits set | First to be evicted |

> 💡 **Guaranteed** pods get the highest scheduling priority and are last evicted under memory pressure. Always use for critical workloads like databases.

---

## 🎯 Scheduling Strategies

```bash
# Label your nodes first
kubectl label node <worker-1> disktype=ssd
kubectl label node <worker-2> disktype=hdd

# Pin a pod to a specific labeled node
kubectl apply -f nginx/nginx_deploy_nodeselector.yaml

# Advanced: required vs preferred node rules
kubectl apply -f nginx/nginx_deploy_nodeaff.yaml

# Co-locate pods (e.g., app + cache on same node)
kubectl apply -f nginx/nginx_deploy_podaff.yaml

# Spread pods across nodes (high availability)
kubectl apply -f nginx/nginx_deploy_podantiaff.yaml

# Observe where pods land
kubectl get pods -o wide
```

### When to Use What?

| Strategy | Use Case | Example |
|----------|----------|---------|
| `NodeSelector` | Simple node targeting | GPU nodes, SSD nodes |
| `NodeAffinity` (hard) | Strict placement rules | Region/zone pinning |
| `NodeAffinity` (soft) | Preferred placement | Cost optimization |
| `PodAffinity` | Co-locate for performance | App + cache same node |
| `PodAntiAffinity` | Spread for resilience | 3 replicas, 3 nodes |
| `Tolerations` | Schedule on tainted nodes | Dedicated node pools |

---

## 🧪 Step-by-Step Lab Guide

### 🥇 Phase 1 — Build Your Kubernetes Cluster

```bash
# On all nodes
sudo apt-get update
sudo apt-get install -y containerd kubelet kubeadm kubectl
sudo systemctl disable --now swap

# On control plane only
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

# Install Calico CNI
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# Verify
kubectl get nodes
```

---

### 🥈 Phase 2 — Install AWS CSI Drivers

```bash
# EBS CSI Driver
helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
helm install aws-ebs-csi-driver aws-ebs-csi-driver/aws-ebs-csi-driver -n kube-system

# EFS CSI Driver
helm repo add aws-efs-csi-driver https://kubernetes-sigs.github.io/aws-efs-csi-driver
helm install aws-efs-csi-driver aws-efs-csi-driver/aws-efs-csi-driver -n kube-system

# Verify CSI pods are running
kubectl get pods -n kube-system | grep csi
```
> ⚠️ Create your EFS Filesystem manually in AWS Console first. Copy the `fs-xxxxxxxx` ID into `efs-sc.yaml`.

---

### 🥉 Phase 3 — Storage Testing

```bash
# Test 1: Static HostPath
kubectl apply -f volumes/hostpath/pv.yaml && kubectl apply -f volumes/hostpath/pvc.yaml
kubectl get pv,pvc

# Test 2: Dynamic EBS
kubectl apply -f volumes/dynamic-ebs/ebs-sc.yaml
kubectl apply -f volumes/dynamic-ebs/ebs-test-pvc.yaml
kubectl apply -f volumes/dynamic-ebs/ebs-test-pod.yaml
kubectl exec -it <ebs-pod> -- df -h   # confirm mount

# Test 3: Shared EFS
kubectl apply -f volumes/dynamic-efs/efs-sc.yaml
kubectl apply -f volumes/dynamic-efs/efs-test-pvc.yaml
kubectl apply -f volumes/dynamic-efs/efs-test-pod.yaml
kubectl apply -f volumes/dynamic-efs/efs-test-pod-2.yaml
# Write in pod-1, read in pod-2 — the RWX magic!
```

---

### 🏅 Phase 4 — PostgreSQL with Secrets + StatefulSet

```bash
# Apply resource governance first
kubectl apply -f volumes/secrets/postgres-secret/limitrange.yaml
kubectl apply -f volumes/secrets/postgres-secret/resourcequota.yaml

# Deploy PostgreSQL StatefulSet
kubectl apply -f volumes/secrets/postgres-secret/statefulset.yaml

# Test both ways of consuming secrets
kubectl apply -f volumes/secrets/postgres-secret/postgres-secret-env-pod.yaml
kubectl apply -f volumes/secrets/postgres-secret/postgres-secret-vol-pod.yaml

# Test Guaranteed QoS
kubectl apply -f volumes/secrets/postgres-secret/guaranteed-pod.yaml
kubectl describe pod guaranteed-pod | grep "QoS Class"
```

---

### 🏅 Phase 5 — Jenkins Deployment

```bash
# Option A: Simple deployment
kubectl apply -f jenkins/jenkins-deployment.yaml
kubectl apply -f jenkins/jenkins-service.yaml

# Option B: Persistent Jenkins with EFS (survives restarts!)
kubectl apply -f volumes/dynamic-efs/jenkins-efs-test.yaml

# Get the external URL
kubectl get svc jenkins-service
# Visit http://<EXTERNAL-IP>:8080
```

---

### 🏅 Phase 6 — Scheduling Experiments

```bash
kubectl apply -f nginx/nginx_deploy_nodeselector.yaml
kubectl get pods -o wide   # All pods on ssd node?

kubectl apply -f nginx/nginx_deploy_podantiaff.yaml
kubectl get pods -o wide   # Pods spread across nodes?
```

---

## 🐛 Troubleshooting Guide

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `PVC Pending` | CSI driver not running | `kubectl get pods -n kube-system \| grep csi` |
| `PVC Pending` | Wrong StorageClass name | `kubectl get sc` and match name in PVC |
| `PVC Pending` (EFS) | Wrong `fileSystemId` | Check `efs-sc.yaml` has correct `fs-xxxxx` |
| `CrashLoopBackOff` | Wrong volume mount path | `kubectl logs <pod>` for details |
| `LoadBalancer Pending` | AWS LB controller missing | Install `aws-load-balancer-controller` |
| `Pod Unschedulable` | No nodes match affinity | `kubectl get nodes --show-labels` |
| Port `2049` blocked | EFS NFS port closed | Add inbound rule for port 2049 in SG |
| Pod evicted under pressure | BestEffort QoS | Set equal `requests == limits` for Guaranteed QoS |
| Secret not found | Wrong secret name/key | `kubectl get secret <n> -o yaml` |

---

## ✅ Lab Completion Checklist

```
Cluster Setup
 □ kubeadm cluster running on EC2
 □ Calico CNI installed
 □ All nodes in Ready state

Storage
 □ HostPath PV/PVC working
 □ EBS dynamic provisioning working
 □ EFS RWX shared across 2 pods
 □ Jenkins persisting data to EFS

Databases & Secrets
 □ PostgreSQL StatefulSet deployed
 □ Secret consumed as env vars
 □ Secret consumed as volume mount
 □ Guaranteed QoS pod verified

Resource Management
 □ ResourceQuota applied to namespace
 □ LimitRange defaults verified
 □ Pod QoS class confirmed

Workloads
 □ Jenkins Deployment + Service live
 □ Nginx DaemonSet running on all nodes
 □ Tomcat behind AWS LoadBalancer

Scheduling
 □ NodeSelector pod placed correctly
 □ NodeAffinity hard rule tested
 □ PodAntiAffinity spreading replicas

Config
 □ ConfigMap injected as env vars
 □ Private registry pull working
```

---

## 🧠 Production Best Practices

- 🚫 **Never** run stateful apps as bare Pods — always use Deployments or StatefulSets
- 💾 **EFS** for shared/Jenkins workloads, **EBS** for databases — never mix them up
- 🔐 **IRSA** (IAM Roles for Service Accounts) over static credentials for AWS access
- 🌐 Use **Ingress + ALB** instead of NodePort for prod HTTP traffic
- 🎯 Always set **resource requests and limits** — or one noisy pod can starve the node
- 📋 **Label everything** — nodes, pods, namespaces. Scheduling and debugging depend on it
- ⚖️ Use **Guaranteed QoS** for critical workloads (databases, Jenkins) — they survive memory pressure
- 📊 Apply **ResourceQuotas per namespace** in multi-team clusters to prevent resource monopolization
- 🔒 Never hardcode secrets in YAML — use Kubernetes Secrets or AWS Secrets Manager

---

## 👨‍💻 Author

<div align="center">

**Vinayak**

*DevOps Engineer | Kubernetes Practitioner | AWS Enthusiast*

[![GitHub](https://img.shields.io/badge/GitHub-vinayak432-181717?style=for-the-badge&logo=github)](https://github.com/vinayak432)

---

*Built with ☕, AWS bills, and many hours of `kubectl describe pod`*

⭐ **Star this repo** if it helped you level up your Kubernetes game!

</div>
