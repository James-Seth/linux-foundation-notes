# Chapter 7: Container Orchestration

## Table of Contents

1. [Kubernetes Overview](#kubernetes-overview)
2. [Kubernetes Architecture](#kubernetes-architecture)
   - [Core Components](#core-components)
   - [Key API Resources](#key-api-resources)
3. [Kubernetes Key Features](#kubernetes-key-features)
4. [Kubernetes Benefits](#kubernetes-benefits)
5. [Managed Kubernetes Services](#managed-kubernetes-services)
   - [Cloud Providers](#cloud-providers)
   - [Other Managed Solutions](#other-managed-solutions)
   - [Platform Solutions (Vendor Lock-in Free)](#platform-solutions-vendor-lock-in-free)
6. [Docker Swarm](#docker-swarm)
7. [HashiCorp Nomad](#hashicorp-nomad)
8. [Amazon ECS](#amazon-ecs)
9. [Azure Container Instances (ACI)](#azure-container-instances-aci)
10. [Summary](#summary)
11. [Glossary](#glossary)

## Glossary

**ACI (Azure Container Instances)** - Microsoft Azure's simplified container deployment service without infrastructure management

**AKS (Azure Kubernetes Service)** - Microsoft's fully-managed Kubernetes service on Azure

**API Gateway** - Entry point for all client requests to access backend services (kube-apiserver in Kubernetes)

**Auto-scaling** - Automatic adjustment of computing resources based on demand and resource utilization

**Bin-packing** - Resource allocation strategy that efficiently uses available resources by placing workloads optimally

**CNCF (Cloud Native Computing Foundation)** - Non-profit organization that hosts cloud-native open source projects like Kubernetes

**Container Runtime** - Software responsible for running containers (containerd, CRI-O, Docker Engine)

**Control Plane** - Set of components that manage the Kubernetes cluster and make global decisions

**DaemonSet** - Kubernetes controller ensuring one Pod replica runs on each cluster node

**Deployment** - Kubernetes controller providing declarative updates for Pods and ReplicaSets

**Docker Swarm** - Docker's native container orchestration platform

**ECS (Elastic Container Service)** - Amazon's fully managed container orchestration service

**EKS (Elastic Kubernetes Service)** - Amazon's managed Kubernetes service on AWS

**etcd** - Distributed key-value store used as Kubernetes' backing store for cluster data

**Fargate** - AWS serverless compute engine for containers (no server management required)

**GKE (Google Kubernetes Engine)** - Google's fully-managed Kubernetes service on Google Cloud Platform

**HCL (HashiCorp Configuration Language)** - Declarative language used for HashiCorp tools like Nomad

**High Availability (HA)** - System design ensuring continuous operation through redundancy and failover

**kube-apiserver** - Kubernetes API server that exposes the Kubernetes API

**kube-controller-manager** - Kubernetes component running controller processes

**kube-proxy** - Network proxy running on each node, implementing part of the Kubernetes Service concept

**kube-scheduler** - Kubernetes component that selects nodes for newly created Pods

**kubelet** - Primary node agent that communicates with the Kubernetes API server

**Labels** - Key-value pairs attached to Kubernetes objects for identification and grouping

**Load Balancing** - Distribution of incoming network traffic across multiple targets

**Namespace** - Virtual cluster within a Kubernetes cluster for resource isolation

**Nomad** - HashiCorp's distributed cluster manager and scheduler

**Orchestration** - Automated coordination and management of containerized applications

**Pod** - Smallest deployable unit in Kubernetes, containing one or more containers

**RBAC (Role-Based Access Control)** - Method of regulating access based on individual user roles

**Raft Consensus** - Algorithm used for distributed consensus in systems like Docker Swarm

**ReplicaSet** - Kubernetes controller maintaining a stable set of replica Pods

**Rolling Update** - Deployment strategy that gradually replaces old instances with new ones

**Scheduler** - Component responsible for assigning workloads to appropriate nodes

**Selector** - Mechanism to identify and group Kubernetes objects based on labels

**Self-healing** - System's ability to automatically detect and recover from failures

**Service** - Kubernetes abstraction defining a logical set of Pods and access policy

**Service Discovery** - Mechanism for services to find and communicate with each other

**Sidecar Pattern** - Design pattern where auxiliary containers run alongside main application containers

**StatefulSet** - Kubernetes controller for managing stateful applications

**Task Definition** - Blueprint specifying how containers should run in Amazon ECS

**TLS (Transport Layer Security)** - Cryptographic protocol for secure communication

**Volume** - Kubernetes abstraction for persistent storage accessible to containers

**Worker Node** - Machine (physical or virtual) that runs containerized applications in Kubernetes

**YAML (YAML Ain't Markup Language)** - Human-readable data serialization standard used for Kubernetes configurations

## Kubernetes Overview

**Definition**: Open-source platform for automating deployment, operations, and scaling of containerized applications

**History**:
- Started by Google (2014)
- Transferred to CNCF (July 2015)
- Apache 2.0 licensed

**Supported Container Runtimes**: containerd, CRI-O, Docker Engine, Mirantis Container Runtime

## Kubernetes Architecture

### Core Components

#### Cluster
- Collection of systems (bare-metal/virtual) running containerized applications

#### Control-Plane Node
- **Purpose**: Scheduling decisions, worker node management, access control, state reconciliation
- **Components**:
  - `kube-apiserver`: API gateway
  - `etcd`: Distributed key-value store
  - `kube-scheduler`: Pod scheduling
  - `kube-controller-manager`: State reconciliation
- **HA**: Multiple control-plane nodes for redundancy

#### Worker Node
- **Purpose**: Run containerized workloads in Pods
- **Components**:
  - `kubelet`: Node agent (container lifecycle management)
  - `kube-proxy`: Network proxy for service access
  - Container runtime

#### Namespace
- Logical cluster partitioning for multi-tenancy
- Resource isolation for projects/teams/environments

### Key API Resources

#### Pod
- **Definition**: Smallest deployable unit
- **Purpose**: Co-location of containers with shared dependencies
- **Limitations**: No self-healing, scaling, or rolling updates
- **Best Practice**: Use controllers (ReplicaSet, Deployment) to manage Pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    run: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.17.9
    ports:
    - containerPort: 80
```

#### ReplicaSet
- **Purpose**: Maintains desired number of Pod replicas
- **Features**: State reconciliation, self-healing

#### Deployment
- **Purpose**: Declarative updates for Pods and ReplicaSets
- **Features**: Rolling updates, rollbacks, scaling, pause/resume

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.9
        ports:
        - containerPort: 80
```

#### DaemonSet
- **Purpose**: Ensures one Pod replica per cluster node
- **Use Cases**: Node agents, monitoring, logging

#### Service
- **Purpose**: Load-balanced access to Pod groups
- **Features**: DNS registration, cluster-internal static IP

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: nginx-deployment
    tier: frontend
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 80
  selector:
    app: nginx-deployment
    tier: frontend
```

#### Labels & Selectors
- **Labels**: Key-value pairs for resource tagging
- **Selectors**: Query mechanism for grouping labeled resources

#### Volumes
- **Purpose**: Persistent storage abstraction
- **Types**: Local, network, distributed, cloud storage

## Kubernetes Key Features

- **Auto-distribution**: Containers distributed based on resource requirements and constraints
- **Horizontal Scaling**: CLI/UI based + auto-scaling on resource utilization
- **Rolling Updates/Rollbacks**: Zero-downtime deployments
- **Storage Integration**: AWS, Azure, GCP, VMware, NFS, iSCSI, CephFS support
- **Self-healing**: Restart failed containers, reschedule from failed nodes
- **Configuration Management**: Secrets and ConfigMaps without image rebuilds
- **Batch Execution**: Support for batch jobs
- **High Availability**: Control-plane resilience
- **Portability**: Infrastructure agnostic

## Kubernetes Benefits

- **Open Source**: Complete orchestration, service discovery, load balancing
- **Scale Management**: Multi-container management at scale
- **Automation**: Deployment, scaling, operations automation
- **Portability**: On-premises and multi-cloud support
- **Consistency**: Same environment across dev/test/prod
- **Resource Efficiency**: Optimal resource utilization

## Managed Kubernetes Services

### Cloud Providers

#### Amazon EKS
- **Service**: Managed Kubernetes on AWS
- **Features**:
  - AWS-managed control plane
  - Multi-AZ high availability
  - Auto-scaling worker nodes
  - IAM + RBAC integration
  - AWS services integration

#### Azure AKS
- **Service**: Fully-managed Kubernetes on Azure
- **Features**:
  - Free control plane
  - GUI + CLI deployment
  - Azure Active Directory integration
  - SOC/ISO/HIPAA/HITRUST compliance

#### Google GKE
- **Service**: Fully-managed Kubernetes on GCP
- **Features**:
  - Container-optimized OS
  - Private container registry
  - Google Cloud operations integration
  - Auto Repair for unhealthy nodes
  - 99.5% SLA
  - Hybrid networking support

### Other Managed Solutions

- **IBM Cloud Kubernetes Service**: Multi-zone, multi-region clusters
- **Oracle OKE**: Enterprise-grade on Oracle Cloud Infrastructure
- **Red Hat OpenShift**: Multi-cloud managed Kubernetes
- **Vultr VKE**: Integrated with Vultr cloud services
- **VMware Tanzu**: Multi-cloud enterprise Kubernetes

### Platform Solutions (Vendor Lock-in Free)

- **Canonical Managed Kubernetes**: Multi-cloud with monitoring/analytics
- **D2iQ DKP**: Enterprise platform with DC/OS features
- **Kubermatic**: Automated cluster management
- **Mirantis Kubernetes Engine**: Cross-infrastructure management
- **Platform9 PMK**: SaaS Kubernetes management
- **Rackspace MPK**: Cross-cloud portable solution
- **Rancher RKE**: Container-based distribution

## Docker Swarm

### Overview
- **Definition**: Native Docker container orchestration
- **Architecture**: Logical grouping of Docker Engines into clusters

### Components

#### Swarm Manager Nodes
- Accept cluster commands
- Make scheduling decisions
- Maintain cluster state (Raft consensus)
- Active/passive HA configuration

#### Swarm Worker Nodes
- Run Docker Engine
- Execute container workloads from managers

### Key Features
- **Docker Compatibility**: Works with existing Docker tools/API
- **Native Networking/Volumes**: Built-in Docker support
- **Scalability**: Large node clusters
- **High Availability**: Manager failover support
- **Declarative**: Define desired service state
- **Security**: TLS-enforced node communication
- **Rolling Updates**: Controlled deployment with rollback

### Solutions
- **Swarm Mode**: Built into Docker Engine
- **Mirantis Container Cloud**: Enterprise Swarm + Kubernetes
- **Mirantis Kubernetes Engine**: Mixed-mode orchestration

### Benefits
- **Native Integration**: Seamless Docker workflow
- **Easy Setup**: Straightforward configuration
- **Scalability**: High availability support
- **Efficiency**: Reduced deployment/management overhead

## HashiCorp Nomad

### Overview
- **Definition**: Distributed cluster manager and resource scheduler
- **Workloads**: Docker containers, VMs, Java applications
- **Platforms**: Linux, Windows, Mac (on-premises + cloud)

### Sample Job File
```hcl
job "hashicorp/web/frontend" {
    region = "us"
    datacenters = ["us-west-1", "us-east-1"]
    
    constraint {
        attribute = "$attr.kernel.name"
        value = "linux"
    }
    
    update {
        stagger = "30s"
        max_parallel = 1
    }
    
    group "frontend" {
        count = 10
        
        task "web" {
            driver = "docker"
            config {
                image = "hashicorp/web-frontend:latest"
            }
            
            resources {
                cpu = 500
                memory = 128
                network {
                    mbits = 10
                    dynamic_ports = ["http"]
                }
            }
        }
    }
}
```

### Key Features
- **Cluster Management**: Combined management and scheduling
- **Multi-workload**: Containers, VMs, unikernels, Java apps
- **Multi-datacenter**: Cross-cloud cluster support
- **Resource Optimization**: Bin-packing for high utilization
- **Scale**: Millions of containers deployment
- **Dry Run**: Preview scheduling actions
- **Self-healing**: Automatic failure recovery
- **Rolling Upgrades**: Built-in deployment strategies
- **Kubernetes Integration**: Multi-orchestrator patterns
- **HashiCorp Ecosystem**: Terraform, Consul, Vault integration
- **Deployment Strategies**: Blue-green, canary deployments

### Benefits
- **Single Binary**: Simple distribution
- **High Availability**: Scalable architecture
- **Resource Efficiency**: Maximized utilization
- **Multi-cloud**: Federation support
- **Zero Downtime**: Maintenance operations
- **Large Scale**: 10,000+ node clusters

## Amazon ECS

### Overview
- **Service**: Fully managed container orchestration on AWS
- **Features**: Fast, secure, highly scalable

### Launch Types

#### Fargate
- **Model**: Serverless containers
- **Management**: AWS handles infrastructure
- **Configuration**: CPU, memory, networking, IAM only

#### EC2
- **Model**: Self-managed instances
- **Control**: Full server customization
- **Responsibility**: Provision, patch, scale clusters

#### External (ECS Anywhere)
- **Model**: On-premises container management
- **Integration**: Hybrid cloud infrastructure

### Core Components

#### Cluster
- Logical grouping of tasks/services
- Container instance grouping (EC2 mode)

#### Task Definition
- Application blueprint specification
- Multi-container configuration

```json
{
  "containerDefinitions": [
    {
      "name": "wordpress",
      "image": "wordpress",
      "portMappings": [{"containerPort": 80, "hostPort": 80}],
      "memory": 500,
      "cpu": 10,
      "links": ["mysql"]
    },
    {
      "name": "mysql",
      "image": "mysql",
      "environment": [{"name": "MYSQL_ROOT_PASSWORD", "value": "password"}],
      "memory": 500,
      "cpu": 10
    }
  ],
  "family": "hello_world"
}
```

#### Service
- Manages task instances based on task definition
- Elastic Load Balancer integration
- Auto-restart unhealthy tasks

#### Scheduler
- Places tasks on cluster nodes

#### Task
- Running container instance from task definition

### Key Features
- **Container Support**: Docker + Windows containers
- **Managed Cluster**: No cluster management overhead
- **JSON Configuration**: Task definition files
- **API Management**: Full API coverage
- **Easy Updates**: Container version management
- **Monitoring**: CloudWatch integration
- **Logging**: CloudTrail integration
- **Registry Support**: Docker Hub, ECR, third-party
- **AWS Fargate**: Serverless container execution
- **Security**: VPC, Security Groups, IAM integration
- **High Availability**: Multi-AZ deployment
- **AWS Integration**: ELB, VPC, IAM, ECR, Batch, CloudWatch

### Benefits
- **Managed Infrastructure**: Public/private/hybrid cloud
- **EC2 Foundation**: Built on proven AWS compute
- **High Availability**: Scalable architecture
- **AWS Ecosystem**: Leverages CloudWatch and other services
- **Multiple Interfaces**: CLI, Dashboard, API management

## Azure Container Instances (ACI)

### Overview
- **Service**: Simplified container deployment without infrastructure management
- **Use Case**: Simple applications, automated tasks, jobs
- **Limitation**: Basic scheduling (use AKS for advanced features)
- **Integration**: Can work with AKS in layered approach

### Key Features
- **Direct Internet Access**: IP addresses + FQDN
- **Container Interaction**: Shell access to running containers
- **VM-like Isolation**: Application-level isolation
- **Resource Specification**: CPU and memory allocation
- **Persistent Storage**: Azure File shares mounting
- **OS Support**: Linux and Windows containers
- **Container Groups**: Single and multi-container scheduling
- **Design Patterns**: Sidecar pattern support

## Summary

Container orchestration platforms provide essential capabilities for managing containerized applications at scale:

- **Kubernetes**: Most popular, feature-rich, cloud-native standard
- **Docker Swarm**: Simple, Docker-native orchestration
- **Nomad**: Multi-workload, HashiCorp ecosystem integration
- **Amazon ECS**: AWS-native with Fargate serverless option
- **Azure ACI**: Simple container deployment for basic use cases

Choose based on:
- **Complexity requirements**: Simple (Swarm/ACI) vs Advanced (Kubernetes)
- **Cloud provider**: Native services (EKS/AKS/GKE) vs Platform-agnostic
- **Workload types**: Containers-only vs Multi-workload (Nomad)
- **Management preference**: Fully-managed vs Self-managed