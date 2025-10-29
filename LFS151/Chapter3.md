# Chapter 3: Infrastructure as a Service (IaaS)

## Table of Contents
1. [IaaS Overview](#iaas-overview)
2. [Amazon EC2](#amazon-ec2)
3. [Azure Virtual Machines](#azure-virtual-machines)
4. [DigitalOcean Droplets](#digitalocean-droplets)
5. [Google Compute Engine](#google-compute-engine)
6. [IBM Cloud Virtual Servers](#ibm-cloud-virtual-servers)
7. [Linode Compute Instances](#linode-compute-instances)
8. [Oracle Cloud Compute](#oracle-cloud-compute)
9. [OpenStack](#openstack)
10. [Vultr Cloud Compute](#vultr-cloud-compute)
11. [Summary](#summary)
12. [Glossary](#glossary)

## IaaS Overview

**Definition**: Cloud service model providing virtualized computing resources over the internet

**Core Components**:
- Virtual machines (compute)
- Storage systems
- Network infrastructure
- Load balancers
- Security groups

**Key Benefits**:
- On-demand provisioning
- Pay-as-you-go pricing
- Global availability
- Scalability
- No hardware management

## Amazon EC2

**Provider**: Amazon Web Services (AWS)
**Hypervisors**: Xen, KVM, Nitro
**Free Tier**: Available with limitations

**Instance Families**:
- **General Purpose**: Balanced CPU/memory (t3, m5, m6i)
- **Compute Optimized**: High CPU performance (c5, c6i)
- **Memory Optimized**: High RAM (r5, x1e, z1d)
- **Storage Optimized**: High I/O (i3, d2, h1)
- **Accelerated Computing**: GPU/FPGA (p4, g4, f1)

**Key Features**:
- AMIs for quick deployment
- Auto Scaling groups
- Elastic Load Balancing
- VPC networking
- CloudWatch monitoring

## Azure Virtual Machines

**Provider**: Microsoft Azure
**Hypervisor**: Azure Hypervisor (customized Hyper-V)
**Free Tier**: 12-month free services

**VM Size Families**:
- **General Purpose**: A, B, D, DC series
- **Compute Optimized**: F series
- **Memory Optimized**: E, M, G series
- **Storage Optimized**: L series
- **GPU Optimized**: N series

**Key Features**:
- Azure Marketplace images
- VM Scale Sets
- Managed Disks
- Virtual Networks
- Azure Monitor

## DigitalOcean Droplets

**Provider**: DigitalOcean
**Hypervisor**: KVM
**Focus**: Developer simplicity

**Droplet Types**:
- **Shared CPU**: Basic, cost-effective
- **Dedicated CPU**: Guaranteed performance
- **Memory-Optimized**: High RAM ratios

**Key Features**:
- SSD storage standard
- One-click applications
- Simple pricing
- Global data centers
- Team collaboration

## Google Compute Engine

**Provider**: Google Cloud Platform
**Hypervisor**: KVM
**Free Tier**: Always-free tier + $300 credit

**Machine Types**:
- **General Purpose**: E2, N2, N1 series
- **Compute Optimized**: C2 series
- **Memory Optimized**: M2, M1 series
- **Accelerator Optimized**: A2 series

**Key Features**:
- Custom machine types
- Live migration
- Sustained use discounts
- Preemptible instances
- Global load balancing

## IBM Cloud Virtual Servers

**Provider**: IBM Cloud
**Hypervisors**: Multiple options (z/VM, PowerVM, XenServer, VMware)

**Server Types**:
- **Public**: Multi-tenant, cost-effective
- **Dedicated**: Single-tenant, enhanced security
- **Transient**: Discounted, interruptible
- **Reserved**: Committed term pricing

**Profile Categories**:
- Balanced, Compute, Memory, Storage, GPU

## Linode Compute Instances

**Provider**: Linode (Akamai Connected Cloud)
**Focus**: Developer-friendly, cost-effective

**Plan Types**:
- **Shared CPU**: Nanode, Standard
- **Dedicated CPU**: Guaranteed resources
- **High Memory**: Memory-optimized
- **GPU**: NVIDIA RTX 6000

## Oracle Cloud Compute

**Provider**: Oracle Cloud
**Measurement**: Oracle Compute Units (OCPUs)

**Instance Shapes**:
- **Standard**: General-purpose computing
- **Dense I/O**: High-performance local storage
- **GPU**: ML and graphics workloads

**Key Features**:
- Bare metal support
- Nested virtualization
- Always Free Tier
- Oracle software optimization

## OpenStack

**Type**: Open-source cloud platform
**License**: Apache 2.0
**Governance**: OpenInfra Foundation

**Core Services**:
- **Nova**: VM provisioning
- **Swift**: Object storage
- **Cinder**: Block storage
- **Neutron**: Networking
- **Keystone**: Identity management
- **Horizon**: Web dashboard

**Deployment**: Complex installation, enterprise focus

## Vultr Cloud Compute

**Provider**: Vultr
**Focus**: Developer and business simplicity

**Instance Categories**:
- **Regular**: Shared vCPU
- **High Frequency**: Dedicated vCPU, 3+ GHz
- **CPU Optimized**: High CPU-to-memory ratio
- **Memory Optimized**: High RAM
- **Storage Optimized**: NVMe SSD
- **Bare Metal**: Single-tenant servers

## Summary

**Market Leaders**:
- **AWS EC2**: Most comprehensive, extensive ecosystem
- **Azure VMs**: Windows integration, hybrid cloud
- **Google Compute Engine**: Innovation, performance

**Specialized Providers**:
- **DigitalOcean**: Developer-focused, simple pricing
- **Linode**: Cost-effective, excellent documentation
- **IBM Cloud**: Enterprise heritage, compliance
- **Oracle Cloud**: Database optimization, free tier
- **Vultr**: Simplicity, global network

**Open Source**:
- **OpenStack**: Private cloud, vendor independence

**Decision Factors**:
- **Workload**: General purpose vs specialized
- **Budget**: Compare pricing models
- **Compliance**: Security requirements
- **Integration**: Ecosystem compatibility
- **Support**: SLA and support levels

**Best Practices**:
- Start small, scale as needed
- Monitor costs actively
- Implement proper security
- Plan for disaster recovery
- Use automation tools

**Recommendation**: Choose based on requirements - AWS/Azure for enterprise, Google Cloud for ML/innovation, DigitalOcean/Linode for development, OpenStack for private cloud control.

## Glossary

**AMI (Amazon Machine Image)** - Pre-configured virtual machine templates for AWS EC2 instances

**Auto Scaling** - Automatic adjustment of compute resources based on demand patterns

**Availability Zone** - Physically separate data center locations within a region

**Bare Metal** - Physical servers without virtualization layer

**CloudWatch** - AWS monitoring and observability service

**Compute Engine** - Google Cloud's infrastructure service for virtual machine provisioning

**Dedicated CPU** - Physical CPU cores allocated exclusively to a single virtual machine

**Droplet** - DigitalOcean's term for virtual machine instances

**EC2 (Elastic Compute Cloud)** - Amazon's virtual machine service within AWS

**GPU Instance** - Virtual machines with dedicated graphics processing units

**Hypervisor** - Software layer managing virtual machines on physical hardware

**IaaS (Infrastructure as a Service)** - Cloud service model providing virtualized computing resources

**Instance** - Individual virtual machine running on cloud infrastructure

**KVM** - Kernel-based Virtual Machine hypervisor used by many cloud providers

**OpenStack** - Open-source cloud computing platform for building private/public clouds

**Region** - Geographic area containing multiple availability zones

**Security Group** - Virtual firewall controlling inbound and outbound traffic

**Virtual Machine** - Software emulation of physical computer system

**VPC (Virtual Private Cloud)** - Isolated virtual network environment within public cloud
