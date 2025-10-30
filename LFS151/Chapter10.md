# Chapter 10: Software-Defined Networking (SDN) and Container Networking

## Table of Contents
1. [SDN Overview](#sdn-overview)
2. [SDN Architecture](#sdn-architecture)
3. [Container Networking Standards](#container-networking-standards)
4. [Service Discovery](#service-discovery)
5. [Docker Networking](#docker-networking)
6. [Kubernetes Networking](#kubernetes-networking)
7. [Cloud Foundry Networking](#cloud-foundry-networking)

## SDN Overview

**Definition**: Software-defined networking decouples the network control layer from the traffic forwarding layer

**Problem Statement**: Traditional networks cannot handle demands from:
- Mobile devices
- Cloud computing
- Distributed architectures (VMs, containers, microservices)

**Solution**: Virtualize networking to provide flexibility matching compute and storage virtualization

**Key Benefit**: Programmable control layer enables custom rules for new networking requirements

## SDN Architecture

### Network Planes

| Plane | Function | Components |
|-------|----------|------------|
| **Data Plane** (Forwarding) | Handle data packets, apply actions based on lookup tables | Routers, switches, modems |
| **Control Plane** | Calculate and program forwarding decisions, implement QoS/VLANs | Routing protocols, device discovery |
| **Management Plane** | Configure, monitor, manage network devices | SNMP, configuration tools |

### SDN Framework

**Traditional**: Control and Data planes tightly coupled
**SDN**: Control and Data planes decoupled

**Benefits**:
- **Centralized Control**: Global network view
- **Programmable**: Well-defined APIs for applications
- **Flexible**: Use tools like Ansible/Chef for configuration
- **Standard Protocols**: OpenFlow for Control-to-Data plane communication
## Container Networking Standards

### Container Network Model (CNM)
**Driver**: Docker, Inc. (libnetwork project)

**Components**:
- **Network Sandbox**: Isolated network environment
- **Endpoints**: Connection points
- **Networks**: Virtual networks

**Modes**:
- **Null**: No networking (NOOP implementation)
- **Bridge**: Linux bridging implementation
- **Overlay/Swarm**: Multi-host communication via VXLAN
- **Remote**: Third-party driver support

### Container Networking Interface (CNI)
**Driver**: Cloud Native Computing Foundation (CNCF)

**Purpose**: 
- Configure network interfaces in Linux containers
- Remove allocated resources when container deleted

**Used By**: Kubernetes, OpenShift, Cloud Foundry

**Scope**: Limited to network connectivity (focused approach)
## Service Discovery

**Definition**: Mechanism enabling processes/services to find and communicate with each other automatically

**Purpose**: Map container names to IP addresses for location-independent access

### Two-Step Process

#### 1. Registration
- Container scheduler registers container name-to-IP mapping
- Stored in key-value store (etcd, Consul)
- Updated when container restarts/stops

#### 2. Lookup
- Services retrieve IP addresses by container name
- Supported by local DNS services
- DNS resolves requests using key-value store data

**DNS Solutions**: Consul, CoreDNS, SkyDNS, Mesos-DNS
## Docker Networking

### Single-Host Drivers

| Driver | Purpose | Characteristics |
|--------|---------|-----------------|
| **Bridge** | Default networking | Linux bridge (docker0), unique IP per container, vEth pairs |
| **Host** | Share host network | Full host network access, security concerns |
| **None** | No networking | Loopback interface only, isolated |

### Bridge Driver Details
```bash
# Default networks
docker network ls
# NETWORK ID    NAME     DRIVER
# 6f30debc...   bridge   bridge
# a1798169...   host     host  
# 4eb0210d...   none     null

# Create custom bridge
docker network create --driver bridge my_bridge

# Use custom network
docker container run --net=my_bridge -itd --name=c2 busybox
```

### Multi-Host Drivers

#### Docker Overlay Driver
- **Method**: VXLAN-based encapsulation
- **Process**: Encapsulate container packets in host packets
- **Use Case**: Docker Swarm networking

#### Macvlan Driver
- **Method**: Assign MAC addresses to containers
- **Appearance**: Containers appear as physical devices
- **Requirement**: Hardware support needed
- **Benefit**: Direct container-to-container communication

### Third-Party Plugins
- **Kuryr**: OpenStack Neutron integration
- **Weave Net**: Multi-host networking with service discovery
## Kubernetes Networking

### Container Networking Model

| Component | Purpose | Implementation |
|-----------|---------|----------------|
| **Pods** | Basic networking unit | Share network namespace |
| **Services** | Stable network identity | Virtual IP with load balancing |
| **Ingress** | External access | L7 load balancer for HTTP(S) |

### Networking Requirements

1. **Pod-to-Pod Communication**
   - All pods communicate without NAT
   - Containers within pod share localhost
   - Flat network address space

2. **Service Discovery**
   - DNS-based service resolution
   - Environment variables (legacy)
   - Label selectors for pod targeting

### Kubernetes Network Plugins (CNI)

#### Popular CNI Plugins

| Plugin | Focus | Capabilities |
|--------|-------|-------------|
| **Flannel** | Simplicity | L3 overlay network, VXLAN backend |
| **Calico** | Security | BGP routing, network policies |
| **Weave Net** | Ease of use | Automatic mesh, built-in DNS |
| **Cilium** | Advanced | eBPF-based, API-aware security |

#### CNI Configuration Example
```yaml
# /etc/cni/net.d/10-flannel.conf
{
  "name": "cbr0",
  "type": "flannel",
  "delegate": {
    "hairpinMode": true,
    "isDefaultGateway": true
  }
}
```

### Service Types

| Type | Purpose | Access Method |
|------|---------|---------------|
| **ClusterIP** | Internal only | Cluster internal IP |
| **NodePort** | External via nodes | High port on all nodes |
| **LoadBalancer** | Cloud integration | External load balancer |
| **ExternalName** | External service mapping | DNS CNAME record |

## Cloud Provider CNI Plugins

### AWS VPC CNI
- **Purpose**: Integrated AWS VPC networking
- **Features**: VPC flow logs, routing policies, security groups
- **Benefit**: Native AWS integration
Azure CNI for Kubernetes

Azure CNI is an open source plugin that integrates Kubernetes Pods with an Azure Virtual Network (also known as VNet) providing network performance at par with VMs.
Calico

Calico uses the BGP protocol to meet the Kubernetes networking requirements.
Cilium

Cilium provides secure network connectivity between application containers. It is L7/HTTP aware and can also enforce network policies on L3-L7.
Flannel

Flannel uses the overlay network, as we have seen with Docker, to meet the Kubernetes networking requirements.
NSX-T

NSX-T from VMware provides network virtualization for a multi-cloud and multi-hypervisor environment. The NSX-T Container Plug-in (NCP) provides integration between NSX-T and container orchestrators such as Kubernetes.
OVN-Kubernetes

OVN-Kubernetes provides overlay based networking for Kubernetes, and Open VSwitch (OVS) based load balancing and network policy.
Weave Net

Weave Net, a simple network for Kubernetes, may run as a CNI plug-in or stand-alone. It does not require additional configuration to run, and the network provides the one IP address per pod, as it is required and expected by Kubernetes.
Multi-Node Networking with Kubernetes

Let's illustrate how microservices communicate on a multi-node Kubernetes cluster.

NOTE: The following set of screenshots may have a different look on your system. The Kubernetes platform is constantly evolving and features may change without notice.

On a two worker node Kubernetes cluster the Pod network is managed by the Calico network plugin. Two distinct application Pods are deployed, a client application and a server application, each deployed to a node. The listing displays the two application Pods with their respective IP addresses and the nodes hosting them. The Service exposing the webserver application displays a private virtual ClusterIP and its Endpoint.

 

Screenshot showing the kubectl -n demos get po,svc,ep -l demo=demo3 -owide command

 

The Pods deployed to different nodes can communicate with each other over the Pod network managed by the network plugin. From the client application we can submit a curl request targeting the IP address of the webserver application Pod. However, the Pods and their respective IP addresses are considered ephemeral resources, thus unreliable for long term communication.

 

Screenshot of the kubectl -n demos exec client-app-7d9bb6c977-fslxq — sh -c ‘curl -s 192.168.142.153’ command

 

The Service exposing the webserver application Pod is also accessible from the client application Pod by targeting the Service ClusterIP. However, this ClusterIP may also change in time even if considered less ephemeral in comparison with the application Pod IP addresses.

 

Screenshot showing the kubectl -n demos exec client-app-7d9bb6c977-fslxq — sh -c ‘curl -s 10.98.67.45’ command

 

The Cluster DNS server helps to resolve communication issues between application Pods hosted by different nodes.

 

Screenshot of the kubectl -n demos exec client-app-7d9bb6c977-fslxq --sh -c ‘curl -s web-app-svc command

 

## Cloud Foundry Networking

### Container-to-Container Communication

| Component | Purpose | Function |
|-----------|---------|----------|
| **Gorouter** | Default routing | External/internal traffic routing |
| **C2C Networking** | Direct communication | Bypass Gorouter for app-to-app |

### CF Networking Components

#### Network Policy Server
- **Role**: Management node
- **Function**: Database of app traffic policies
- **Scope**: Cluster-wide policy management

#### Garden External Networker
- **Method**: CNI plugin integration
- **Purpose**: App network setup and exposure
- **Access**: Gorouter, TCP Router, SSH Proxy traffic

#### Silk CNI Plugin
- **Technology**: VXLAN overlay network
- **IP Management**: Unique IP per container
- **Security**: Non-routable overlay prevents traffic escape

#### VXLAN Policy Agent
- **Function**: Network policy enforcement
- **Rules**: Source app, destination app, protocol, ports
- **Bypass**: Direct communication without Gorouter

### Network Policy Configuration
```yaml
# Example network policy
source: app-frontend
destination: app-backend
protocol: tcp
ports: 8080
```

---

## Summary

This chapter covered SDN and container networking fundamentals:

- **SDN Architecture**: Control/data plane separation for centralized management
- **Container Networking Standards**: CNM vs CNI approaches and implementations
- **Service Discovery**: DNS and API-based service location mechanisms
- **Docker Networking**: Single-host and multi-host networking drivers
- **Kubernetes Networking**: Pod networking, CNI plugins, and service types
- **Cloud Foundry Networking**: Container-to-container communication and policy enforcement

## Key Takeaways

1. **SDN** provides centralized network control and programmability
2. **Container networking** requires standardized interfaces (CNM/CNI)
3. **Service discovery** enables dynamic service location in distributed systems
4. **Multiple solutions** exist for different platforms and use cases
5. **Security policies** are essential for container-to-container communicationTrim the fluff and turn it into notes