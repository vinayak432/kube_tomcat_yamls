Kubernetes on AWS – Jenkins & Tomcat Deployment Repo
📌 Overview

This repository contains Kubernetes manifests for deploying:

Jenkins (multiple deployment patterns)

Tomcat application

AWS EBS dynamic provisioning

AWS EFS dynamic provisioning

AWS Cloud Controller Manager configuration

Services (ClusterIP / LoadBalancer)

ReplicaSets, Deployments, DaemonSets

Persistent Volumes (static & dynamic)

This setup is designed for a kubeadm-based Kubernetes cluster running on AWS EC2.

🏗 Architecture Diagram (ASCII)
                           ┌──────────────────────────┐
                           │        AWS Cloud         │
                           └──────────────────────────┘
                                       │
               ┌────────────────────────────────────────────┐
               │                    VPC                     │
               └────────────────────────────────────────────┘
                         │                         │
                ┌────────┴────────┐        ┌───────┴────────┐
                │  Control Plane  │        │   Worker Node  │
                │  (kubeadm)      │        │  (EC2)         │
                ├──────────────────┤        ├────────────────┤
                │ kube-apiserver  │        │ kubelet        │
                │ etcd            │        │ containerd     │
                │ controller-mgr  │        │ calico-node    │
                └──────────────────┘        └────────────────┘
                                                      │
                                      ┌───────────────┼───────────────┐
                                      │                               │
                               ┌─────────────┐                ┌─────────────┐
                               │   EBS CSI   │                │   EFS CSI   │
                               └─────────────┘                └─────────────┘
                                      │                               │
                               ┌─────────────┐                ┌─────────────┐
                               │  EBS Volume │                │  EFS FS     │
                               │  (RWO)      │                │  (RWX)      │
                               └─────────────┘                └─────────────┘
                                      │                               │
                               ┌─────────────┐                ┌─────────────┐
                               │  Jenkins    │                │  Tomcat     │
                               └─────────────┘                └─────────────┘
🔄 Deployment Flow Diagram
Developer → kubectl apply
            ↓
API Server validates YAML
            ↓
Scheduler selects Worker Node
            ↓
PVC Created
            ↓
CSI Driver Provisions Volume
            ↓
PV Bound to PVC
            ↓
Pod Scheduled
            ↓
Container Started
            ↓
Service Exposed (ClusterIP / LoadBalancer)
            ↓
AWS Load Balancer Created (if type=LoadBalancer)

🏗 Architecture

Kubernetes installed using kubeadm

Calico CNI for networking

AWS Load Balancer Controller

EBS CSI Driver (for block storage)

EFS CSI Driver (for shared storage)

📁 Repository Structure

🔹 Storage Configuration
EBS (Block Storage)

ebs-sc.yaml → StorageClass for EBS dynamic provisioning

ebs-test-pvc.yaml → PVC using EBS

ebs-test-pod.yaml → Test pod mounting EBS volume

Use case:

Single-node ReadWriteOnce storage

Stateful apps needing block storage

EFS (Shared Storage)

efs-sc.yaml → StorageClass for EFS (dynamic Access Points)

efs-test-pvc.yaml → PVC using EFS

efs-test-pod.yaml → Test pod mounting EFS

efs-test-pod-2.yaml → Multi-pod test for RWX validation

jenkins-efs-test.yaml → Jenkins using EFS backend

Use case:

Shared ReadWriteMany storage

Multi-pod shared data

Jenkins home directory (production style)

⚠ Important:
EFS File System must be created manually in AWS before dynamic provisioning works.

🔹 Static Volume Examples

pv.yaml → Static PersistentVolume

pvc.yaml → PVC bound to static PV

pvc_mnt.yaml → Pod mounting static PVC

Use case:

Learning PV/PVC binding

Non-dynamic environments

🔹 Jenkins Deployments

Multiple Jenkins deployment methods included:

jenkins-pod.yaml → Single pod

jenkins-rs.yaml → ReplicaSet

jenkins-deployment.yaml → Deployment

jenkins-daemonset.yaml → DaemonSet

jen_host.yaml → HostPath-based Jenkins

jenkins-service.yaml → Service exposure

Deployment options demonstrated:

Method	Purpose
Pod	Basic understanding
ReplicaSet	Pod management
Deployment	Production approach
DaemonSet	Per-node instance
HostPath	Local node storage
EFS	Shared persistent storage
🔹 Tomcat Application

Inside tomcat/ directory:

tomcat_pod.yaml

tomcat-rs.yaml

tomcat-deployment.yaml

tomcat_service_lb.yaml

Demonstrates:

Web application deployment

Replica scaling

AWS LoadBalancer service

🔹 AWS Cloud Components

aws-ccm-values.yaml → AWS Cloud Controller Manager configuration

Used when:

Integrating Kubernetes with AWS infrastructure

Managing LoadBalancers and node metadata

🚀 Setup Guide
1️⃣ Prerequisites

EC2 instances (Control Plane + Worker)

IAM role attached to nodes:

AmazonEBSCSIDriverPolicy

AmazonEFSCSIDriverPolicy

Security groups allowing:

6443 (API Server)

2049 (EFS NFS)

Node-to-node traffic

2️⃣ Install Storage Drivers
EBS CSI Driver

Required for dynamic EBS provisioning.

EFS CSI Driver

Required for dynamic EFS provisioning.
EFS File System must exist in AWS.

3️⃣ Apply StorageClass
kubectl apply -f ebs-sc.yaml
kubectl apply -f efs-sc.yaml
4️⃣ Create PVC
kubectl apply -f ebs-test-pvc.yaml
kubectl apply -f efs-test-pvc.yaml

Verify:

kubectl get pvc

Status should be:

Bound
5️⃣ Deploy Applications

Example:

kubectl apply -f jenkins-deployment.yaml
kubectl apply -f jenkins-service.yaml

Check pods:

kubectl get pods -owide
🧪 Testing Storage
EBS Test
kubectl apply -f ebs-test-pod.yaml

Exec inside pod and create file to verify persistence.

EFS Test (Multi-Pod)
kubectl apply -f efs-test-pod.yaml
kubectl apply -f efs-test-pod-2.yaml

Create file in one pod → verify visibility in second pod.

🌐 Accessing Applications

If using LoadBalancer service:

kubectl get svc

Copy EXTERNAL-IP once assigned.

📊 Learning Objectives Covered

kubeadm cluster setup

Static vs Dynamic PV

EBS vs EFS storage differences

RWX vs RWO access modes

HostPath vs CSI drivers

ReplicaSets vs Deployments

AWS integration with Kubernetes

Cloud Controller Manager concepts

⚠ Common Issues & Fixes
Problem	Cause	Fix
PVC Pending	No CSI driver	Install driver
PVC Pending (EFS)	No EFS File System	Create in AWS
Pod CrashLoop	Permission issue	Fix volume ownership
LoadBalancer pending	No AWS controller	Install LB controller
🎯 Production Recommendations

Use EFS for Jenkins home

Use EBS for databases

Avoid HostPath in production

Use Deployment instead of Pod/RS

Enable IAM roles for service accounts (IRSA)

Use Ingress + ALB instead of NodePort
