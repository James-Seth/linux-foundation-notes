# Chapter 14: Infrastructure as Code Tools

## Table of Contents

1. [Infrastructure as Code Overview](#infrastructure-as-code-overview)
2. [Terraform](#terraform)
   - Security Best Practices
   - Terraform Best Practices
3. [AWS CloudFormation](#aws-cloudformation)
   - Security Best Practices
   - CloudFormation Best Practices
4. [BOSH](#bosh)
   - Security Best Practices
   - BOSH Best Practices
5. [Tool Comparison](#tool-comparison)
6. [Security and Best Practices Overview](#security-and-best-practices-overview)
7. [Glossary](#glossary)
8. [Summary](#summary)

---

## Infrastructure as Code Overview

### The Concept

**Infrastructure as Code (IaC)**: Defining infrastructure in version-controlled configuration files instead of manual setup.

**Key Benefit**: Create reproducible deployment environments across development, staging, and production.

**Version Control Integration**: Combining IaC with versioned software guarantees reproducible build and release environments every time.

**Multi-Cloud Support**: Same infrastructure definitions can deploy to different cloud providers with appropriate tooling.

---

## Terraform

### Overview

**Developer**: HashiCorp

**Purpose**: Define infrastructure as code for deployment on VMs, bare metal, or cloud platforms.

**Configuration Language**: HashiCorp Configuration Language (HCL)

**Enterprise Offerings**:
- **Terraform Cloud**: Collaboration platform for teams
- **Terraform Enterprise**: Advanced enterprise features and support

### What Terraform Manages

**Resources**: Physical machines, VMs, network switches, containers, and more.

**Infrastructure Types**:
- Virtual machines
- Bare metal servers
- Cloud infrastructure
- Network components
- Storage systems

---

## Terraform Providers

### Definition

**Provider**: Plugin responsible for understanding API interactions and exposing resources to Terraform.

**Purpose**: Makes Terraform agnostic to underlying platforms.

**Customization**: Custom providers can be created through plugins.

### Provider Categories

**IaaS (Infrastructure as a Service)**:
- AWS (Amazon Web Services)
- GCP (Google Cloud Platform)
- Azure (Microsoft Azure)
- DigitalOcean
- OpenStack
- Alibaba Cloud
- Oracle Cloud

**PaaS (Platform as a Service)**:
- Heroku
- Cloud Foundry

**SaaS (Software as a Service)**:
- DNSimple
- Cloudflare
- Datadog

---

## Key Features

### 1. Infrastructure as Code

- Infrastructure described using high-level configuration syntax
- Datacenter blueprint can be versioned like application code
- Configurations can be shared and reused across teams
- Enables collaboration through version control

### 2. Execution Plans

**Planning Step**: Terraform generates execution plan before making changes.

**Purpose**: Shows exactly what Terraform will do when you run apply.

**Benefit**: No surprises or unexpected infrastructure changes.

### 3. Resource Graph

**Graph Building**: Terraform creates a dependency graph of all resources.

**Parallelization**: Creates and modifies non-dependent resources in parallel.

**Benefits**:
- Efficient infrastructure provisioning
- Clear visibility into resource dependencies
- Optimal execution order

### 4. Change Automation

**Complex Changes**: Apply complex changesets with minimal human interaction.

**Predictability**: Execution plan and resource graph show exactly what will change and in what order.

**Error Reduction**: Automation reduces human errors in infrastructure changes.

---

## Terraform Workflow

### Required Files

**main.tf**: Declares infrastructure resources to be provisioned.

**outputs.tf**: Formats output returned after provisioning (e.g., URLs, IP addresses).

### Basic Commands

**Initialize Plugins**:
```bash
terraform init
```
- Initializes plugins necessary for provisioning
- Downloads provider plugins
- Sets up backend configuration

**Plan (Dry Run)**:
```bash
terraform plan
```
- Shows what changes will be made
- No actual infrastructure changes
- Review before applying

**Apply Configuration**:
```bash
terraform apply
```
- Provisions declared resources
- Requires "yes" confirmation at prompt
- Creates or updates infrastructure

**Destroy Resources**:
```bash
terraform destroy
```
- Removes all provisioned resources
- Requires "yes" confirmation at prompt
- Useful for cleanup or testing

### Example Workflow

**Step 1**: Create configuration files (`main.tf`, `outputs.tf`).

**Step 2**: Initialize Terraform.
```bash
terraform init
```

**Step 3**: Review planned changes.
```bash
terraform plan
```

**Step 4**: Apply configuration.
```bash
terraform apply
```

**Step 5**: Verify provisioned resources (web browser, cloud console, curl command).

**Step 6**: Destroy resources when done (optional).
```bash
terraform destroy
```

**Note**: Terraform CLI features may change without notice. Commands may produce slightly different outputs on your system.

### Complete Example: AWS Web Server

This example demonstrates a complete Terraform configuration for deploying a web server on AWS.

**File: main.tf**
```hcl
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "webserver/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "Terraform"
      Project     = "WebServer"
    }
  }
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
}

# Public Subnet
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.subnet_cidr
  availability_zone       = var.availability_zone
  map_public_ip_on_launch = true
}

# Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

# Security Group
resource "aws_security_group" "web" {
  name        = "web-server-sg"
  description = "Security group for web server"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "HTTP from anywhere"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS from anywhere"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "SSH from specific IP"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = var.admin_ips
  }

  egress {
    description = "Allow all outbound"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# EC2 Instance
resource "aws_instance" "web" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web.id]
  key_name               = var.key_name

  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              echo "<h1>Hello from Terraform</h1>" > /var/www/html/index.html
              EOF

  root_block_device {
    volume_size = 20
    volume_type = "gp3"
    encrypted   = true
  }

  lifecycle {
    create_before_destroy = true
  }
}

# Elastic IP
resource "aws_eip" "web" {
  instance = aws_instance.web.id
  domain   = "vpc"

  depends_on = [aws_internet_gateway.main]
}
```

**File: variables.tf**
```hcl
variable "aws_region" {
  description = "AWS region to deploy resources"
  type        = string
  default     = "us-west-2"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "subnet_cidr" {
  description = "CIDR block for subnet"
  type        = string
  default     = "10.0.1.0/24"
}

variable "availability_zone" {
  description = "Availability zone for subnet"
  type        = string
  default     = "us-west-2a"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "ami_id" {
  description = "AMI ID for EC2 instance"
  type        = string
  # Example AMI - Replace with current Amazon Linux 2023 AMI for your region
  # Use: aws ec2 describe-images --owners amazon --filters "Name=name,Values=al2023-ami-*" --query 'Images[0].ImageId'
  default     = "ami-0123456789abcdef0"
}

variable "key_name" {
  description = "SSH key pair name"
  type        = string
}

variable "admin_ips" {
  description = "List of admin IP addresses for SSH access (CIDR notation)"
  type        = list(string)
  default     = []
  sensitive   = true
}
```

**File: outputs.tf**
```hcl
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}

output "instance_public_ip" {
  description = "Public IP address of the instance"
  value       = aws_eip.web.public_ip
}

output "instance_private_ip" {
  description = "Private IP address of the instance"
  value       = aws_instance.web.private_ip
}

output "security_group_id" {
  description = "ID of the security group"
  value       = aws_security_group.web.id
}

output "web_url" {
  description = "URL to access the web server"
  value       = "http://${aws_eip.web.public_ip}"
}
```

**File: terraform.tfvars (example - DO NOT commit this file)**
```hcl
aws_region        = "us-west-2"
environment       = "dev"
key_name          = "my-ssh-key"
admin_ips         = ["203.0.113.0/32"]  # Replace with your IP(s) in CIDR notation
ami_id            = "ami-0abcdef1234567890"  # Replace with current AMI for your region
```

**Usage**:
```bash
# Initialize
terraform init

# Format code
terraform fmt

# Validate
terraform validate

# Plan
terraform plan -out=tfplan

# Apply
terraform apply tfplan

# Check outputs
terraform output web_url

# Destroy when done
terraform destroy
```

### Security Best Practices

**1. State File Security**:
- Never commit state files to version control (contains sensitive data)
- Use remote state backends with encryption (S3, Terraform Cloud, Azure Storage)
- Enable state locking to prevent concurrent modifications
- Restrict access to state files with IAM policies

**Example - Secure remote backend**:
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

**2. Secrets Management**:
- Never hardcode credentials in `.tf` files
- Use environment variables for sensitive data
- Leverage cloud provider secret managers (AWS Secrets Manager, Azure Key Vault)
- Consider HashiCorp Vault integration
- Mark outputs as sensitive when they contain secrets

**Example - Sensitive variables**:
```hcl
variable "db_password" {
  type      = string
  sensitive = true
}

output "connection_string" {
  value     = "connection string here"
  sensitive = true
}
```

**3. Access Control**:
- Use least privilege principle for provider credentials
- Implement RBAC in Terraform Cloud/Enterprise
- Separate credentials by environment (dev, staging, prod)
- Rotate credentials regularly
- Use service accounts instead of personal accounts

**4. Code Security**:
- Use `.gitignore` to exclude sensitive files
- Scan for hardcoded secrets with tools like `tfsec` or `checkov`
- Pin provider versions to prevent unexpected changes
- Review third-party modules before use
- Enable MFA for Terraform Cloud/Enterprise

**Example - Version pinning**:
```hcl
terraform {
  required_version = "~> 1.5.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

**5. Network Security**:
- Use private subnets for sensitive resources
- Implement security groups with minimal required access
- Enable encryption in transit and at rest
- Use VPNs or private connectivity for state storage

### Terraform Best Practices

**1. Project Structure**:
- Organize code into logical modules
- Separate environments (use workspaces or separate directories)
- Keep configuration DRY (Don't Repeat Yourself)
- Use consistent naming conventions

**Example structure**:
```
terraform/
├── modules/
│   ├── networking/
│   ├── compute/
│   └── database/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
└── global/
```

**2. Module Usage**:
- Create reusable modules for common patterns
- Version modules properly
- Document module inputs and outputs
- Test modules independently
- Use module registry (Terraform Registry, private registry)

**3. State Management**:
- Always use remote state for team environments
- Enable state locking
- Use separate state files per environment
- Back up state files regularly
- Never manually edit state files

**4. Code Quality**:
- Use `terraform fmt` to format code consistently
- Run `terraform validate` before committing
- Use `terraform plan` before every apply
- Implement pre-commit hooks
- Add comments for complex logic

**5. Change Management**:
- Always review execution plans carefully
- Use separate apply steps for critical changes
- Tag resources appropriately for tracking
- Document infrastructure changes
- Implement approval workflows for production

**6. Dependency Management**:
- Use `depends_on` sparingly (Terraform handles most dependencies)
- Understand implicit vs explicit dependencies
- Avoid circular dependencies
- Use data sources to reference external resources

**7. Testing**:
- Test infrastructure changes in non-production first
- Use tools like Terratest for automated testing
- Validate plans before applying
- Implement smoke tests after deployment

**8. Documentation**:
- Document module purpose and usage
- Maintain README files for each module
- Use variable descriptions
- Document architecture decisions
- Keep runbooks for common operations

---

## Benefits

### Key Advantages

**Safe Infrastructure Management**:
- Build, change, and version infrastructure safely and efficiently
- Plan changes before applying them
- Track all infrastructure changes in version control

**Provider Flexibility**:
- Manages existing and customized service providers
- Agnostic to underlying platforms
- Works with multiple cloud providers simultaneously

**Scalability**:
- Manage single application infrastructure
- Manage entire datacenter infrastructure
- Scale from small to large deployments

**Flexible Abstraction**:
- Resources abstracted across different platforms
- Write once, deploy to multiple clouds
- Consistent workflow regardless of provider

**Reproducibility**:
- Same configuration produces identical infrastructure
- Eliminates configuration drift
- Consistent environments across teams

**Collaboration**:
- Infrastructure code can be reviewed like application code
- Team members can collaborate on infrastructure changes
- Version control provides audit trail

---

## AWS CloudFormation

### Overview

**Provider**: Amazon Web Services (AWS)

**Purpose**: Create and manage AWS resources with templates, adding order and predictability to provisioning and updates.

**Template Support**: JSON or YAML format for infrastructure definitions.

**Version Control**: Apply version control to AWS infrastructure similar to software version control.

**Interface Options**:
- AWS CloudFormation Designer (web interface)
- AWS CLI (command-line interface)

### Key Features

**1. Extensibility**:
- Supports modeling, provisioning, and management of third-party app resources
- AWS CloudFormation Registry for monitoring, incident management, and version control

**2. Authoring with JSON or YAML**:
- Model infrastructure and describe resources in text files
- CloudFormation Designer provides visual design assistance

**3. Authoring with Programming Languages**:
- AWS Cloud Development Kit (AWS CDK) supports:
  - TypeScript
  - Python
  - Java
  - .NET
- Integrated with CloudFormation for infrastructure provisioning

**4. Safety Controls**:
- Rollback Triggers to safely revert to prior state
- Automatic rollback on errors

**5. Preview Environment Changes**:
- Model potential impact of proposed changes
- Review changes before applying to existing resources

**6. Dependency Management**:
- Determines action sequence during stack operations
- Handles resource dependencies automatically

**7. Cross-Account and Cross-Region Management**:
- Single template called StackSet
- Deploy to multiple accounts and regions simultaneously

### CloudFormation Workflow

**Basic Commands**:

**Validate Template**:
```bash
aws cloudformation validate-template --template-body file://template.yaml
```
- Validates template syntax before creating resources
- Catches errors early in the process

**Create Stack**:
```bash
aws cloudformation create-stack --stack-name mystack --template-body file://template.yaml
```
- Creates infrastructure stack from template
- Returns stack ID immediately
- Actual resource creation takes longer

**List Exports**:
```bash
aws cloudformation list-exports
```
- Retrieves output values from stacks
- Returns empty if resources not ready yet

**Delete Stack**:
```bash
aws cloudformation delete-stack --stack-name mystack
```
- Removes all stack resources
- Cleanup when resources no longer needed

**Note**: AWS CLI features may change without notice. Commands may produce slightly different outputs on your system.

### Security Best Practices

**1. Template Security**:
- Never hardcode credentials in templates
- Use AWS Secrets Manager or Parameter Store for sensitive data
- Implement IAM roles instead of access keys
- Use `NoEcho` parameter property for sensitive inputs

**Example - Secure parameters**:
```yaml
Parameters:
  DBPassword:
    Type: String
    NoEcho: true
    Description: Database password

Resources:
  DBSecret:
    Type: AWS::SecretsManager::Secret
    Properties:
      Description: Database password
      SecretString: !Ref DBPassword
```

**2. IAM Permissions**:
- Use least privilege for CloudFormation service role
- Separate permissions by environment
- Use IAM policies to restrict stack operations
- Enable CloudTrail for audit logging

**3. Stack Policies**:
- Implement stack policies to prevent accidental updates
- Protect critical resources from deletion
- Use change sets for production stacks

**Example - Stack policy**:
```json
{
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "Update:Delete",
      "Resource": "LogicalResourceId/ProductionDatabase"
    }
  ]
}
```

**4. Encryption**:
- Enable encryption for all data at rest
- Use KMS for encryption key management
- Encrypt S3 buckets storing templates
- Enable encryption for databases and storage

**5. Network Security**:
- Deploy resources in VPCs, not EC2-Classic
- Use security groups with minimal required access
- Implement NACLs for additional layer of security
- Use private subnets for backend resources

### CloudFormation Best Practices

**1. Template Organization**:
- Break large templates into nested stacks
- Use separate templates per environment
- Parameterize for reusability
- Store templates in version control

**2. Change Management**:
- Always use change sets for production updates
- Review change sets before executing
- Test in non-production first
- Implement approval workflows

**3. Error Handling**:
- Configure rollback triggers
- Set appropriate timeouts
- Use DeletionPolicy to protect critical resources
- Monitor stack events during operations

**Example - Deletion policy**:
```yaml
Resources:
  MyDatabase:
    Type: AWS::RDS::DBInstance
    DeletionPolicy: Snapshot
    Properties:
      # ... properties
```

**4. Naming and Tagging**:
- Use descriptive stack names
- Tag all resources consistently
- Include environment and purpose in tags
- Use tags for cost allocation

**5. Drift Detection**:
- Regularly run drift detection
- Remediate drift promptly
- Document intentional out-of-band changes
- Consider using CloudFormation as single source of truth

**6. Cross-Stack References**:
- Use exports for sharing values between stacks
- Minimize dependencies between stacks
- Document cross-stack relationships
- Plan stack deletion order carefully

**7. Validation**:
- Validate templates before deployment
- Use CloudFormation Linter (cfn-lint)
- Test with different parameter combinations
- Verify IAM permissions before deployment

### Benefits of Using CloudFormation

**AWS-Specific Integration**:
- Native AWS service with deep integration
- Supports all AWS resources
- No additional agents or tools required

**Safe Infrastructure Management**:
- Build, change, and version AWS infrastructure safely and efficiently
- Rollback capabilities on failures

**Multiple Authoring Options**:
- Text file configuration (JSON/YAML)
- Programming languages through AWS CDK
- CLI and web interface

**Repeatable Management**:
- Consistent, repeatable infrastructure deployments
- Infrastructure as code principles

**Quick Edits and Version Control**:
- Treat infrastructure like application code
- Track changes through version control systems

---

## BOSH

### Overview

**Definition**: Open source tool for release engineering, deployment, lifecycle management, and monitoring of distributed systems.

**Primary Use Case**: Originally developed to deploy Cloud Foundry PaaS, but can deploy other software (e.g., Hadoop).

**Architecture**: Creates VMs on top of IaaS providers, configures them, and deploys applications.

### Supported IaaS Providers

**Native Support**:
- VMware vSphere
- vCloud Director

**Cloud Provider Interface (CPI) Support**:
- Google Compute Engine
- Amazon AWS
- Microsoft Azure
- OpenStack

### Key Concepts

**1. Stemcell**:
- Versioned, IaaS-specific operating system image
- Pre-installed utilities including BOSH Agent
- No application-specific code
- Released and maintained by BOSH team

**2. Release**:
- Placed on top of a stemcell
- Versioned collection of:
  - Configuration properties
  - Templates
  - Scripts
  - Source code
  - Build instructions for software deployment

**3. Deployment**:
- Collection of VMs built from stemcells
- Populated with specific releases
- Includes disks for persistent data
- Complete application environment

**4. BOSH Director**:
- Central orchestrator component
- Controls VM creation and deployment
- Manages software and service lifecycle events
- Processes deployment manifest files

**Workflow**:
1. Upload stemcells to BOSH Director
2. Upload releases to BOSH Director
3. Upload deployment manifest to BOSH Director
4. Director processes manifest and executes deployment

### Sample Deployment Manifest

Example deployment manifest for ZooKeeper cluster:

```yaml
---
name: zookeeper

releases:
- name: zookeeper
  version: 0.0.5
  url: https://bosh.io/d/github.com/cppforlife/zookeeper-release?v=0.0.5
  sha1: 65a07b7526f108b0863d76aada7fc29e2c9e2095

stemcells:
- alias: default
  os: ubuntu-jammy  # Ubuntu 22.04 LTS (xenial/bionic are EOL)
  version: latest

update:
  canaries: 2
  max_in_flight: 1
  canary_watch_time: 5000-60000
  update_watch_time: 5000-60000

instance_groups:
- name: zookeeper
  azs: [z1, z2, z3]
  instances: 5
  jobs:
  - name: zookeeper
    release: zookeeper
    properties: {}
  vm_type: default
  stemcell: default
  persistent_disk: 10240
  networks:
  - name: default

- name: smoke-test
  azs: [z1]
  lifecycle: errand
  instances: 1
  jobs:
  - name: smoke-tests
    release: zookeeper
    properties: {}
  vm_type: default
  stemcell: default
  networks:
  - name: default
```

**Manifest Components**:
- **name**: Deployment identifier
- **releases**: Software packages to deploy
- **stemcells**: Base OS images to use
- **update**: Update strategy configuration
- **instance_groups**: VM groups with specific roles

### Security Best Practices

**1. Stemcell Security**:
- Always use latest stemcell versions with security patches
- Verify stemcell checksums (sha1) before use
- Use hardened stemcells for production
- Subscribe to security advisories from BOSH team
- Test new stemcells in non-production first

**2. Release Security**:
- Verify release signatures and checksums
- Review release notes for security updates
- Pin release versions in production
- Audit third-party releases before deployment
- Use trusted release sources only

**3. BOSH Director Security**:
- Restrict network access to Director
- Use TLS for all Director communications
- Enable UAA (User Account and Authentication) for access control
- Implement RBAC for team members
- Regular security audits and updates

**4. Secrets Management**:
- Use BOSH CredHub for secrets storage
- Never commit credentials to manifests
- Rotate credentials regularly
- Use variable interpolation for sensitive data
- Encrypt communication between components

**Example - Using CredHub variables**:
```yaml
instance_groups:
- name: web
  jobs:
  - name: webapp
    properties:
      admin_password: ((admin_password))
      db_password: ((db_password))
```

**5. Network Security**:
- Deploy BOSH in private subnets
- Use security groups to restrict traffic
- Enable encryption for inter-VM communication
- Implement network segmentation
- Use VPN for Director access

**6. VM Security**:
- Enable disk encryption
- Configure OS-level firewall rules
- Disable unnecessary services
- Implement log forwarding for monitoring
- Regular security scanning of deployed VMs

### BOSH Best Practices

**1. Deployment Organization**:
- Separate deployments by environment
- Use meaningful deployment names
- Document deployment dependencies
- Maintain deployment topology diagrams

**2. Update Strategies**:
- Use canary deployments for critical updates
- Configure appropriate watch times
- Plan maintenance windows
- Test updates in staging first
- Have rollback plans ready

**Example - Safe update strategy**:
```yaml
update:
  canaries: 1
  max_in_flight: 1
  canary_watch_time: 30000-300000
  update_watch_time: 30000-300000
  serial: true
```

**3. Resource Management**:
- Right-size VMs for workload
- Use persistent disks for stateful services
- Implement proper disk sizing
- Monitor resource utilization
- Clean up orphaned disks regularly

**4. High Availability**:
- Deploy multiple instances for critical services
- Distribute instances across availability zones
- Configure resurrection for failed VMs
- Implement health checks
- Use load balancers for redundancy

**5. Monitoring and Logging**:
- Configure system metrics collection
- Implement centralized logging
- Set up alerting for failures
- Monitor deployment health
- Track stemcell and release versions

**6. Backup and Recovery**:
- Regular backups of BOSH Director
- Backup persistent disks
- Document recovery procedures
- Test disaster recovery plans
- Store backups securely off-site

**7. Manifest Management**:
- Version control all manifests
- Use YAML anchors to reduce duplication
- Validate manifests before deployment
- Document custom properties
- Use ops files for environment-specific changes

**8. Lifecycle Management**:
- Plan for stemcell updates
- Schedule regular release updates
- Maintain deployment history
- Document configuration changes
- Automate routine operations

**9. Errand Usage**:
- Use errands for one-time tasks
- Implement smoke tests as errands
- Clean up with pre-delete errands
- Document errand purposes
- Test errands independently

**10. Cloud Provider Interface (CPI)**:
- Keep CPI updated
- Understand CPI limitations
- Test CPI changes thoroughly
- Monitor CPI-related issues
- Document cloud-specific configurations

### Benefits of Using BOSH

**Release Engineering**:
- Easy identification of release components (source, tools, environment)
- Structured approach to software packaging

**Stability and Reproducibility**:
- Guaranteed consistent deployments
- Versioned releases ensure reproducibility

**Consistent Development Framework**:
- Stable framework for development
- Consistent deployment patterns across environments

**CI/CD Integration**:
- Integrates with continuous integration and deployment pipelines
- Automated testing and deployment workflows

**Multi-Cloud Support**:
- Works across multiple IaaS providers
- Consistent deployment process regardless of cloud platform

**Lifecycle Management**:
- Handles software deployment, updates, and monitoring
- Complete lifecycle from deployment to decommissioning

---

## Tool Comparison

| Feature | Terraform | CloudFormation | BOSH |
|---------|-----------|----------------|------|
| **Provider** | HashiCorp | Amazon Web Services | Open Source (Cloud Foundry) |
| **Cloud Support** | Multi-cloud | AWS only | Multi-cloud (via CPI) |
| **Configuration Language** | HCL | JSON/YAML/CDK | YAML |
| **Primary Use Case** | General IaC | AWS infrastructure | Distributed systems deployment |
| **State Management** | State files | AWS-managed | Director-managed |
| **Planning/Preview** | Yes (terraform plan) | Yes (change sets) | Update strategies |
| **Execution** | CLI/Cloud/Enterprise | CLI/Console/CDK | Director/CLI |
| **Rollback** | Manual/State | Automatic triggers | Update strategies |
| **Version Control** | Built-in | Through Git | Built-in (stemcells/releases) |
| **Best For** | Multi-cloud IaC | AWS-native workflows | Cloud Foundry & distributed apps |

### Choosing the Right Tool

**Choose Terraform if**:
- You need multi-cloud support
- You want provider-agnostic infrastructure
- You need infrastructure for any type of deployment
- You want a unified workflow across clouds

**Choose CloudFormation if**:
- You're exclusively on AWS
- You want native AWS integration
- You need AWS-specific features
- You prefer AWS CDK for programming language support

**Choose BOSH if**:
- You're deploying Cloud Foundry
- You need distributed systems management
- You want complete lifecycle management
- You need structured release engineering

### Real-World Scenarios

**Scenario 1: Multi-Cloud Startup**
- **Challenge**: Deploy application across AWS and GCP, potential Azure expansion
- **Tool**: Terraform
- **Why**: Provider-agnostic approach allows single codebase for multiple clouds. Team learns one tool instead of three. Easy to add new cloud providers.
- **Example**: E-commerce platform with AWS primary, GCP for analytics, considering Azure for specific regional requirements

**Scenario 2: AWS-Native Enterprise**
- **Challenge**: Large enterprise with 100% AWS infrastructure, existing CloudFormation knowledge
- **Tool**: CloudFormation
- **Why**: Native AWS integration, no agent installation, team already trained. AWS CDK allows programming language use. Free service with AWS support.
- **Example**: Financial services company with strict AWS-only policy, existing CloudFormation templates, need for stack policies on production resources

**Scenario 3: Cloud Foundry Platform**
- **Challenge**: Deploy and manage Cloud Foundry PaaS across multiple data centers
- **Tool**: BOSH
- **Why**: Built specifically for Cloud Foundry, handles complex distributed system deployments. Stemcells provide consistent OS layer. Built-in lifecycle management and monitoring.
- **Example**: Large organization providing internal PaaS to development teams, needs automated updates and health monitoring

**Scenario 4: Hybrid Multi-Cloud with Kubernetes**
- **Challenge**: On-premises data center plus AWS and Azure, running Kubernetes
- **Tool**: Terraform
- **Why**: Manages infrastructure across all environments. Can provision Kubernetes clusters on any platform. Modules provide reusable patterns.
- **Example**: Healthcare provider with on-prem systems, AWS for patient portal, Azure for analytics, consistent Kubernetes deployment across all

**Scenario 5: AWS Migration Project**
- **Challenge**: Migrating legacy applications to AWS, need infrastructure documentation
- **Tool**: CloudFormation
- **Why**: CloudFormation templates document infrastructure as code. Native AWS service means no third-party dependencies. Can model existing resources.
- **Example**: Government agency migrating from physical data center to AWS GovCloud, needs full audit trail and compliance documentation

**Scenario 6: Microservices Platform**
- **Challenge**: Deploy and manage hundreds of microservices with consistent patterns
- **Tool**: BOSH or Terraform (depends on platform)
- **Why BOSH**: If using Cloud Foundry for microservices. Built-in service discovery, health checks, and resurrection.
- **Why Terraform**: If using Kubernetes or mixed container platforms. Service mesh (Istio/Linkerd) handled by Terraform modules.
- **Example**: Streaming media company with 300+ microservices, need automated deployments, rollbacks, and canary releases

---

## Security and Best Practices Overview

### Common Security Principles Across All Tools

**1. Secrets Management**:
- Never hardcode credentials in configuration files
- Use dedicated secret management services
- Rotate credentials regularly
- Implement encryption at rest and in transit

**2. Access Control**:
- Apply least privilege principle
- Use role-based access control (RBAC)
- Separate credentials by environment
- Enable multi-factor authentication
- Regular access audits

**3. Code Security**:
- Version control all infrastructure code
- Exclude sensitive files from repositories
- Scan code for security vulnerabilities
- Pin versions to prevent supply chain attacks
- Code review process for changes

**4. Network Security**:
- Use private networks for sensitive resources
- Implement proper firewall rules
- Enable encryption for all communications
- Use VPNs for remote access
- Network segmentation by environment

**5. Monitoring and Auditing**:
- Enable audit logging for all operations
- Monitor infrastructure for drift
- Set up alerts for security events
- Regular security assessments
- Maintain incident response plans

### Common Best Practices Across All Tools

**1. Environment Separation**:
- Separate development, staging, and production
- Use different accounts/projects per environment
- Isolated credentials per environment
- Test changes in non-production first

**2. Change Management**:
- Review all changes before applying
- Use preview/plan features
- Implement approval workflows for production
- Document all infrastructure changes
- Maintain rollback procedures

**3. Documentation**:
- Document architecture and design decisions
- Maintain runbooks for common operations
- Keep README files updated
- Document disaster recovery procedures
- Track infrastructure dependencies

**4. Testing**:
- Validate configurations before deployment
- Implement automated testing where possible
- Test disaster recovery procedures
- Conduct regular security testing
- Load testing for critical systems

**5. Backup and Recovery**:
- Regular backups of state/configuration
- Test recovery procedures
- Document recovery time objectives (RTO)
- Off-site backup storage
- Version backup strategies

**6. Continuous Improvement**:
- Regular security updates
- Stay current with best practices
- Learn from incidents
- Refactor and optimize regularly
- Team training and skill development

### Tool-Specific Security Focus

**Terraform**:
- State file protection is critical (contains sensitive data)
- Use remote backends with encryption and locking
- Leverage provider-specific security features
- Regular security scanning with tfsec/checkov

**CloudFormation**:
- Stack policies for production resources
- IAM role-based deployment
- Change sets for all production changes
- CloudTrail for complete audit trail

**BOSH**:
- CredHub for centralized secrets management
- Stemcell security updates critical
- UAA for Director authentication
- Network isolation for Director

### Compliance Considerations

**1. Regulatory Requirements**:
- Understand applicable regulations (GDPR, HIPAA, SOC2)
- Implement required controls
- Maintain audit trails
- Regular compliance audits
- Document compliance measures

**2. Industry Standards**:
- Follow CIS benchmarks
- Implement security frameworks (NIST, ISO 27001)
- Regular security assessments
- Penetration testing
- Security awareness training

**3. Data Protection**:
- Classify data by sensitivity
- Encrypt sensitive data
- Implement data retention policies
- Secure data disposal
- Privacy by design

---

## Common Pitfalls and Solutions

### Terraform Pitfalls

**Pitfall 1: State File Conflicts**
- **Problem**: Multiple team members running terraform apply simultaneously, corrupting state
- **Solution**: Always use remote state backend with locking (S3 + DynamoDB, Terraform Cloud)
- **Prevention**: Never use local state for team projects

**Pitfall 2: Accidental Resource Destruction**
- **Problem**: Running terraform destroy or terraform apply with wrong workspace/directory
- **Solution**: Use lifecycle prevent_destroy, separate workspaces, add confirmation prompts
- **Prevention**: Always run terraform plan first, use -target flag for specific resources

**Example - Prevent destruction**:
```hcl
resource "aws_db_instance" "production" {
  # ... configuration ...
  
  lifecycle {
    prevent_destroy = true
  }
}
```

**Pitfall 3: Hardcoded Credentials**
- **Problem**: Credentials committed to Git repository
- **Solution**: Use environment variables, AWS profiles, or secret management tools
- **Prevention**: Add *.tfvars to .gitignore, use pre-commit hooks to scan for secrets

**Pitfall 4: Provider Version Changes**
- **Problem**: Provider updates break existing code
- **Solution**: Always pin provider versions in required_providers block
- **Prevention**: Test provider updates in non-production first

**Pitfall 5: Large State Files**
- **Problem**: State file becomes huge, slow operations
- **Solution**: Split infrastructure into multiple state files by environment or component
- **Prevention**: Don't put everything in one root module

### CloudFormation Pitfalls

**Pitfall 1: Stack Stuck in UPDATE_ROLLBACK_FAILED**
- **Problem**: Stack update fails, rollback also fails, stack is stuck
- **Solution**: Use continue-update-rollback with skip-resources-in-rollback for problematic resources
- **Command**: `aws cloudformation continue-update-rollback --stack-name mystack --resources-to-skip LogicalResourceId`

**Pitfall 2: Resource Limit Exceeded**
- **Problem**: Template has more than 500 resources (CloudFormation limit)
- **Solution**: Use nested stacks to break template into smaller pieces
- **Prevention**: Plan stack boundaries before reaching limit

**Example - Nested stack**:
```yaml
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/bucket/network.yaml
      Parameters:
        VPCCidr: 10.0.0.0/16
```

**Pitfall 3: Circular Dependencies**
- **Problem**: Resources depend on each other in a loop
- **Solution**: Use DependsOn attribute carefully, redesign resource relationships
- **Prevention**: Draw dependency graph before writing template

**Pitfall 4: Manual Changes Causing Drift**
- **Problem**: Someone changes resources via console, stack out of sync
- **Solution**: Run drift detection regularly, import resources back into stack
- **Prevention**: Enforce all changes through CloudFormation, use stack policies

**Pitfall 5: Stack Deletion Failures**
- **Problem**: Cannot delete stack because resources have dependencies
- **Solution**: Use DeletionPolicy: Retain for resources to keep, manually clean up dependencies
- **Prevention**: Design with deletion in mind, document dependency order

### BOSH Pitfalls

**Pitfall 1: Stemcell Version Mismatches**
- **Problem**: Deployment uses different stemcell versions across instances
- **Solution**: Pin stemcell versions in deployment manifest, test updates in staging
- **Prevention**: Use explicit version numbers, not "latest"

**Example - Pinned stemcell**:
```yaml
stemcells:
- alias: default
  os: ubuntu-jammy
  version: "1.537"  # Explicit version
```

**Pitfall 2: Director Out of Disk Space**
- **Problem**: BOSH Director runs out of disk, deployments fail
- **Solution**: Clean up orphaned disks, old releases, unused stemcells
- **Commands**: `bosh clean-up --all`, monitor disk usage
- **Prevention**: Set up automated cleanup jobs, monitor disk space

**Pitfall 3: Network Configuration Errors**
- **Problem**: VMs cannot communicate due to network misconfiguration
- **Solution**: Verify network ranges don't overlap, check security groups/firewall rules
- **Prevention**: Document network topology, use IP address management tool

**Pitfall 4: Failed VM Resurrection**
- **Problem**: BOSH resurrects VM but wrong configuration applied
- **Solution**: Ensure persistent disks are properly configured, check resurrection config
- **Prevention**: Test resurrection behavior in staging

**Pitfall 5: Release Version Conflicts**
- **Problem**: Different deployments need incompatible release versions
- **Solution**: Upload multiple release versions, use explicit version in each deployment
- **Prevention**: Use semantic versioning, maintain compatibility matrices

**Pitfall 6: Update Strategy Too Aggressive**
- **Problem**: Canary update fails but max_in_flight set too high, multiple instances affected
- **Solution**: Start with canaries: 1, max_in_flight: 1 for critical deployments
- **Prevention**: Test update strategies in staging, have rollback plan

### General IaC Pitfalls

**Pitfall 1: No Testing Before Production**
- **Problem**: Deploying untested changes directly to production
- **Solution**: Always test in dev/staging environments first
- **Prevention**: Implement CI/CD pipeline with mandatory testing stages

**Pitfall 2: Lack of Documentation**
- **Problem**: Team members don't understand existing infrastructure
- **Solution**: Document architecture, maintain runbooks, add comments to code
- **Prevention**: Make documentation part of code review process

**Pitfall 3: No Backup Strategy**
- **Problem**: Infrastructure destroyed with no way to recover
- **Solution**: Back up state files, maintain infrastructure diagrams, version control everything
- **Prevention**: Implement automated backup procedures

**Pitfall 4: Ignoring Drift**
- **Problem**: Actual infrastructure diverges from code
- **Solution**: Regular drift detection, import manual changes back into IaC
- **Prevention**: Restrict manual changes, use IaC as single source of truth

**Pitfall 5: Over-Engineering**
- **Problem**: Overly complex abstractions make code hard to understand
- **Solution**: Keep it simple, use modules only when patterns repeat 3+ times
- **Prevention**: Code reviews focusing on simplicity and maintainability

---

## Glossary

**Apply**: Terraform command that provisions or updates infrastructure based on configuration files.

**AWS CDK (Cloud Development Kit)**: Framework for defining AWS infrastructure using programming languages like TypeScript, Python, Java, and .NET.

**AWS CloudFormation**: AWS service for provisioning and managing AWS infrastructure using templates.

**BOSH**: Open source tool for release engineering, deployment, lifecycle management, and monitoring of distributed systems.

**BOSH Agent**: Pre-installed utility on stemcells that communicates with BOSH Director.

**BOSH Director**: Central orchestrator component that controls VM creation, deployment, and lifecycle management in BOSH.

**Canary**: Testing deployment strategy where changes are applied to a small subset of instances first before rolling out to all instances.

**CredHub**: Secure credential storage service used with BOSH for managing secrets and sensitive data.

**Change Set**: CloudFormation feature that previews how proposed changes will affect running resources.

**Checkov**: Open source static analysis tool for scanning infrastructure as code for security issues.

**Cloud Foundry**: Open source Platform as a Service (PaaS) that BOSH was originally designed to deploy.

**Drift Detection**: Feature that identifies differences between actual infrastructure and defined configuration.

**CPI (Cloud Provider Interface)**: BOSH plugin system for supporting different IaaS providers.

**Deployment**: In BOSH, a collection of VMs built from stemcells and populated with releases.

**Deployment Manifest**: YAML file describing the desired state of a BOSH deployment.

**Destroy**: Terraform command that removes all resources defined in the configuration.

**Execution Plan**: Preview of changes Terraform will make before actually applying them.

**HCL (HashiCorp Configuration Language)**: Declarative language used to write Terraform configurations.

**IaC (Infrastructure as Code)**: Practice of managing and provisioning infrastructure through machine-readable definition files.

**IaaS (Infrastructure as a Service)**: Cloud computing model providing virtualized computing resources over the internet.

**Instance Group**: BOSH concept defining a group of VMs with the same configuration and purpose.

**IAM (Identity and Access Management)**: AWS service for managing access to cloud resources securely.

**Init**: Terraform command that initializes working directory and downloads necessary provider plugins.

**JSON (JavaScript Object Notation)**: Text-based data format used for CloudFormation templates.

**KMS (Key Management Service)**: Cloud service for creating and managing encryption keys.

**Least Privilege**: Security principle of granting only minimum necessary permissions.

**Lifecycle Management**: Process of managing software and infrastructure from deployment through updates to decommissioning.

**main.tf**: Primary Terraform configuration file declaring resources to be provisioned.

**NoEcho**: CloudFormation parameter property that masks sensitive data in console and API responses.

**Ops File**: BOSH feature for modifying deployment manifests without changing the base manifest.

**outputs.tf**: Terraform file that defines values to be displayed after provisioning completes.

**PaaS (Platform as a Service)**: Cloud computing model providing a platform for developing, running, and managing applications.

**Plan**: Terraform command that shows what changes will be made without actually applying them.

**Provider**: Plugin that enables Terraform to interact with cloud providers, SaaS providers, and APIs.

**RBAC (Role-Based Access Control)**: Security approach that restricts system access based on user roles.

**Release**: In BOSH, a versioned collection of configuration properties, templates, scripts, and source code for deploying software.

**Release Engineering**: Discipline of building, packaging, and deploying software in a consistent and reproducible manner.

**Remote Backend**: Storage location for Terraform state files hosted on remote services (S3, Terraform Cloud, etc.).

**Resource**: Infrastructure component managed by Terraform (VM, network, storage, etc.).

**Resource Graph**: Dependency graph Terraform builds to determine order of resource creation and modification.

**Resurrection**: BOSH feature that automatically recreates failed VMs.

**Rollback Trigger**: CloudFormation feature that automatically reverts changes if specified conditions are met.

**Stack**: CloudFormation term for a collection of AWS resources managed as a single unit.

**Stack Policy**: CloudFormation feature that prevents unintended updates or deletions of stack resources.

**State**: Terraform's record of currently managed infrastructure, stored in state files.

**State Locking**: Mechanism preventing concurrent Terraform operations that could corrupt state.

**StackSet**: CloudFormation feature for deploying stacks across multiple AWS accounts and regions.

**Stemcell**: In BOSH, a versioned OS image with pre-installed utilities that serves as the base for VMs.

**Template**: Configuration file that defines infrastructure resources (CloudFormation uses JSON/YAML, Terraform uses HCL).

**Terraform Cloud**: HashiCorp's collaboration platform for Terraform teams.

**Terraform Enterprise**: Enterprise version of Terraform with additional features and support.

**Terratest**: Go library for automated testing of infrastructure code.

**Tfsec**: Security scanner for Terraform code that detects potential security issues.

**UAA (User Account and Authentication)**: Authentication and authorization service used with BOSH Director.

**YAML (YAML Ain't Markup Language)**: Human-readable data serialization format used for configuration files.

---

## Summary

This chapter covered three major Infrastructure as Code tools:

**Key Takeaways**:

1. **Infrastructure as Code** enables version-controlled, reproducible infrastructure across all environments.

2. **Terraform** provides multi-cloud infrastructure provisioning with provider-agnostic approach:
   - HCL configuration language
   - Execution planning prevents surprises
   - Resource graph optimizes provisioning
   - Works across AWS, Azure, GCP, and more

3. **AWS CloudFormation** offers native AWS infrastructure management:
   - JSON/YAML templates or AWS CDK programming
   - Deep AWS integration
   - Automatic rollback on failures
   - Cross-account and cross-region deployments

4. **BOSH** specializes in distributed systems and lifecycle management:
   - Stemcells provide versioned OS images
   - Releases package application components
   - BOSH Director orchestrates deployments
   - Strong Cloud Foundry integration

**Decision Factors**:
- **Cloud Strategy**: Single cloud (CloudFormation) vs multi-cloud (Terraform)
- **Use Case**: General infrastructure (Terraform/CloudFormation) vs distributed systems (BOSH)
- **Team Skills**: HCL, JSON/YAML, or programming languages
- **Existing Ecosystem**: AWS native vs cloud-agnostic vs Cloud Foundry

**Best Practices**:
- Store all infrastructure definitions in version control
- Test infrastructure changes in non-production first
- Use planning/preview features before applying changes
- Implement rollback strategies
- Document infrastructure dependencies
- Regular audits of infrastructure state

All three tools enable treating infrastructure as code, providing consistency, reproducibility, and version control for infrastructure deployments.
