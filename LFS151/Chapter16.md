# Chapter 16: Image Creation and Registries

## Table of Contents
- [Overview](#overview)
- [Dockerfiles & Containerfiles](#dockerfiles--containerfiles)
- [Multi-Stage Builds](#multi-stage-builds)
- [Podman & Buildah](#podman--buildah)
- [Packer for VM Images](#packer-for-vm-images)
- [Image Registries](#image-registries)
- [Common Pitfalls](#common-pitfalls)
- [Quick Reference Table](#quick-reference-table)
- [Glossary](#glossary)
- [Summary](#summary)

---

## Overview
- Immutable infrastructure replaces services with new images for updates or fixes.
- Automated image creation ensures consistency and reliability.
- Key tools: Docker, Podman, Buildah (containers); Packer (VMs).

---

## Dockerfiles & Containerfiles
- **Purpose:** Define steps to build container images.
- **Key Instructions:** FROM, RUN, COPY, EXPOSE, CMD.
- **Example Dockerfile:**
  ```dockerfile
  FROM fedora
  RUN dnf -y update && dnf clean all
  RUN dnf -y install nginx && dnf clean all
  RUN echo "daemon off;" >> /etc/nginx/nginx.conf
  RUN echo "nginx on Fedora" > /usr/share/nginx/html/index.html
  EXPOSE 80
  CMD [ "/usr/sbin/nginx" ]
  ```
- **Build Command:**
  - **Docker:** `docker build -t myimage .`
  - **Podman:** `podman build -t myimage .`
- **Best Practice:** Start from a minimal base image (e.g., `alpine`, `scratch`) for smaller, more secure images.

---

## Multi-Stage Builds
- **Purpose:** Reduce image size by separating build and runtime stages.
- **Example:**
  ```dockerfile
  # stage - 1
  FROM ubuntu AS buildstep
  RUN apt update && apt install -y build-essential gcc
  COPY hello.c /app/hello.c
  WORKDIR /app
  RUN gcc -o hello hello.c && chmod +x hello

  # stage - 2
  FROM ubuntu
  WORKDIR /usr/src/app
  COPY --from=buildstep /app/hello ./hello
  CMD ["./hello"]
  ```
- **Note:** The `COPY --from=buildstep` line copies only the built artifact from the previous stage. The C SDK and intermediate artifacts are not copied to the final image.
## Podman & Buildah
- **Podman:** Daemonless container engine, compatible with Docker CLI. Can use Dockerfile or Containerfile.
- **Buildah:** CLI for building images, supports step-by-step commands and Dockerfile/Containerfile.
- **Key Features:**
  - Both tools read instructions from text files (Dockerfile/Containerfile)
  - Identical syntax for both file types
  - More flexible than traditional Docker builds
- **Sample Containerfile:**
  ```dockerfile
  FROM fedora
  RUN dnf -y update && dnf clean all
  RUN dnf -y install nginx && dnf clean all
  RUN echo "daemon off;" >> /etc/nginx/nginx.conf
  RUN echo "nginx on Fedora" > /usr/share/nginx/html/index.html
  EXPOSE 80
  CMD [ "/usr/sbin/nginx" ]
  ``` 

- **Buildah Step-by-Step Example:**
  ```bash
  buildah from fedora
  buildah run containerID -- dnf -y update && dnf clean all
  buildah run containerID -- dnf -y install nginx && dnf clean all
  buildah run containerID -- sh -c 'echo "daemon off;" >> /etc/nginx/nginx.conf'
  buildah run containerID -- sh -c 'echo "nginx on Fedora" > /usr/share/nginx/html/index.html'
  buildah config --port 80 containerID
  buildah config --cmd '/usr/sbin/nginx' containerID
  ```
- **Build Commands:**
  - **Podman:** `podman build -t myimage .`
  - **Buildah:** `buildah bud -t myimage .`
- **Tip:** Use `podman build` or `buildah bud` for Dockerfile/Containerfile builds. 
## Packer for VM Images
- **Purpose:** Automate VM image creation for multiple platforms (AWS, GCP, Azure, VMware, etc.).
- **Config Language:** HCL or JSON.
- **Example:**

  ```json
  {
    "builders": [{
      "type": "amazon-ebs",
      "region": "us-west-2",
      "source_ami": "ami-9abea4fb",
      "instance_type": "t2.micro",
      "ssh_username": "ubuntu",
      "ami_name": "packer-example"
    }],
    "provisioners": [{
      "type": "shell",
      "inline": ["sudo apt-get -y update", "sudo apt-get -y install apache2"]
    }]
  }
  ```
- **Build Command:** `packer build template.json`
- **Best Practice:** Clean up temp files/logs in provisioners to avoid image bloat.
- **Supported Platforms:** AWS, GCP, Azure, VMware, VirtualBox, Docker, and more.
- **Provisioners:** Shell, Ansible, Puppet, Chef.
- **Benefits:**
  - Automates image creation from templates
  - Multi-provider portability
  - Fast infrastructure deployment
  - Consistent, reproducible images

---
## Image Registries
- **Purpose:** Store, share, and manage container/VM images for CI/CD and deployment.
- **Popular Registries:**
  - [Docker Hub](https://hub.docker.com/) - Container image registry for Docker
  - [Harbor](https://goharbor.io/) - CNCF graduated project for container images
  - [Quay](https://quay.io/) - Red Hat registry service with GitHub/Bitbucket integration
  - [Google Container Registry](https://cloud.google.com/container-registry) - Google Cloud service
  - [Amazon ECR](https://aws.amazon.com/ecr/) - AWS service for container images and Helm charts
  - [Azure Container Registry](https://azure.microsoft.com/en-us/products/container-registry/) - Microsoft Azure service
  - [JFrog Artifactory](https://jfrog.com/artifactory/) - Universal artifact manager
  - [ArtifactHub](https://artifacthub.io/) - CNCF project for cloud-native artifacts
  - [Skopeo](https://github.com/containers/skopeo) - Tool for moving images between registries
- **Tip:** Use private registries for sensitive images; tag images with version and environment.

---

## Common Pitfalls
- **Large Image Size:** Not using multi-stage builds or cleaning up after installs.
- **Hardcoded Secrets:** Storing secrets in Dockerfile/Containerfile or image layers is insecure.
- **Missing Context Files:** Build fails if required files are not present in build context.
- **Unpinned Base Images:** Using `latest` tag can cause unpredictable builds.
- **Registry Authentication Issues:** Failing to configure credentials for private registries.
- **Image Drift:** Manual changes to running containers not reflected in source files.
- **VM Image Bloat:** Not cleaning up temp files or logs in Packer provisioners.

---

## Study Questions

1. **What is the main advantage of using multi-stage builds in container image creation?**
2. **How does Podman differ from Docker in terms of architecture?**
3. **What is the purpose of a provisioner in a Packer template?**
4. **Name two common pitfalls when working with image registries.**
5. **Why should you avoid using the `latest` tag for base images?**
6. **What command would you use to build an image with Buildah from a Dockerfile?**
7. **Which registry would you recommend for storing Helm charts and Kubernetes artifacts?**

---

## Quick Reference Table

| Feature         | Docker      | Podman      | Buildah     | Packer      |
|----------------|-------------|-------------|-------------|-------------|
| Build Images    | Yes         | Yes         | Yes         | Yes (VMs)   |
| Daemonless     | No          | Yes         | Yes         | Yes         |
| CLI Compatible | Yes         | Yes         | Yes         | Yes         |
| Multi-Stage    | Yes         | Yes         | Yes         | N/A         |
| VM Images      | No          | No          | No          | Yes         |
| Registry Push  | Yes         | Yes         | Yes         | Yes         |
| Config Format  | Dockerfile  | Dockerfile/Containerfile | Dockerfile/Containerfile | HCL/JSON     |

---

## Glossary

> **📖 Key Terms and Definitions**
> 
> **Immutable Infrastructure:** Infrastructure replaced rather than modified.  
> **Container Image:** Packaged filesystem/configuration for running containers.  
> **Dockerfile/Containerfile:** Instructions to build a container image.  
> **Multi-Stage Build:** Technique to reduce image size by separating build/runtime stages.  
> **Podman:** Daemonless container engine compatible with Docker CLI.  
> **Buildah:** Tool for building OCI container images.  
> **Packer:** Tool for automating VM image creation.  
> **Registry:** Repository for storing/sharing images.  
> **Provisioner:** Packer component for configuring images after build.  
> **Base Image:** Starting point for building a new image.  
> **Context Files:** Files required for building an image (app source, configs).  
> **ArtifactHub:** CNCF project for storing cloud-native artifacts.  
> **Skopeo:** Tool for copying images between registries.

---

## Summary

- Automated image creation is essential for reliable, scalable infrastructure.
- Use Dockerfiles/Containerfiles and multi-stage builds to optimize container images.
- Podman and Buildah offer flexible, open source alternatives to Docker.
- Packer enables fast, repeatable VM image creation for multiple platforms.
- Store images in registries for sharing and CI/CD integration.
- Avoid common pitfalls by following best practices for image size, security, and versioning. 