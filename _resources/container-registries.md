---
layout: page
title: "Container Registries"
permalink: /resources/container-registries/
toc: true
---

* TOC
{:toc}

## Overview

Container registries are centralized repositories that store and distribute container images, enabling you to pull, push, version, and share container images across different systems and platforms. Understanding how to work with registries is essential across all computing environments—from local development to cloud deployments, HPC clusters, and Kubernetes orchestrations. The most common registry format follows the [Open Containers Initiative (OCI)](https://opencontainers.org/) standard, which grew out of Docker's container format. This standardization enables interoperability across different container runtimes and deployment environments.

Registries play a crucial role in:
- **Local Development**: Pulling base images and sharing containers with team members
- **Cloud Environments**: Deploying containers to cloud services like AWS, Azure, and GCP
- **HPC Systems**: Using containers for scientific computing with frameworks like Apptainer, Podman, Shifter, and CharlieCloud
- **Kubernetes Clusters**: Orchestrating containerized applications at scale

## Common Container Registries

### Docker Hub

[Docker Hub](https://hub.docker.com/) is the most widely used public container registry. It hosts millions of public container images and is often the first place to look for pre-built containers.

**Key considerations:**
- Docker Hub introduced API rate limits for anonymous access in November 2020
- For frequent use, it's recommended to pull images to local files rather than repeatedly accessing the registry
- Authentication is required for private containers and to avoid rate limits

### Other Public Registries

- **[Quay.io](https://quay.io/)**: A container registry service that supports both public and private repositories
- **[NVIDIA NGC](https://catalog.ngc.nvidia.com/)**: NVIDIA's container registry specializing in GPU-accelerated applications and deep learning frameworks
- **[GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)**: Integrated with GitHub, allowing you to store containers alongside your source code
- **AWS ECR** (Elastic Container Registry): Amazon's container registry service
- **Azure ACR** (Azure Container Registry): Microsoft's container registry service

## Working with Registries Across Environments

### Local Development

In local development environments, registries are typically accessed using Docker or Podman:

**Docker:**
```bash
docker pull ubuntu:22.04
docker push myregistry.io/myimage:tag
```

**Podman:**
```bash
podman pull docker://ubuntu:22.04
podman push myregistry.io/myimage:tag
```

**Authentication:**
```bash
docker login docker.io
# or
podman login docker.io
```

### Cloud Environments

Cloud providers offer integrated container registries that work seamlessly with their container services:

**AWS ECR (Elastic Container Registry):**
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/myimage:tag
```

**Azure Container Registry (ACR):**
```bash
az acr login --name myregistry
docker push myregistry.azurecr.io/myimage:tag
```

**Google Container Registry (GCR) / Artifact Registry:**
```bash
gcloud auth configure-docker
docker push gcr.io/myproject/myimage:tag
```

### HPC Environments

HPC container frameworks can pull directly from OCI registries, often converting them to native formats:

**Apptainer:**
```bash
apptainer pull docker://sylabsio/lolcow:latest
apptainer registry login --username myuser docker://docker.io
```
The images are cached in `~/.singularity`. The location can be changed via an environment variable `APPTAINER_CACHEDIR`.

**Podman:**
```bash
podman pull docker://ubuntu:22.04
podman login docker.io
```

**Shifter and CharlieCloud** also support pulling from OCI registries, though the exact commands may vary by installation.

**Key HPC Considerations:**
- Pull images to local files (e.g., SIF format) to avoid repeated registry access and rate limits
- HPC systems often have restricted network access, so caching images locally is important
- Some HPC frameworks convert OCI images to their native formats (e.g., Apptainer converts to SIF)

### Kubernetes Environments

Kubernetes pulls container images from registries specified in pod specifications. The kubelet on each node handles image pulling:

**Pod Specification:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: myregistry.io/myimage:tag
    imagePullPolicy: Always
```

**Private Registry Authentication:**

1. **ImagePullSecrets** (per-pod):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  imagePullSecrets:
  - name: regcred
  containers:
  - name: myapp
    image: myregistry.io/myimage:tag
```

2. **ServiceAccount** (cluster-wide):
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-sa
imagePullSecrets:
- name: regcred
```

**Creating ImagePullSecrets:**
```bash
kubectl create secret docker-registry regcred \
  --docker-server=<registry-url> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>
```

**Cloud-Specific Kubernetes Integrations:**
- **AWS EKS**: Can use IAM roles for service accounts (IRSA) to authenticate with ECR
- **Azure AKS**: Can use Azure Active Directory integration for ACR authentication
- **GKE**: Can use Workload Identity to authenticate with GCR/Artifact Registry

### Best Practices

1. **Use Specific Tags**: Always specify version tags rather than using `:latest` to ensure reproducibility across all environments (local, cloud, HPC, Kubernetes).

2. **Pull to Local Files (HPC)**: In HPC environments, pull once to a local file (e.g., SIF format) and reuse it. This avoids rate limits and improves performance on systems with restricted network access.

3. **Cache Management**: 
   - **Local/Cloud**: Use local Docker/Podman caches to speed up builds
   - **HPC**: Clean cache directories periodically to manage disk space on shared systems
   - **Kubernetes**: Configure image pull policies appropriately (`IfNotPresent` vs `Always`)

4. **Private Registries**: For sensitive or proprietary containers, use private registries:
   - **Local**: Run a local registry (e.g., Docker Registry, Harbor)
   - **Cloud**: Use cloud-native registries (ECR, ACR, GCR) with IAM integration
   - **HPC**: Set up on-premises registries or use cloud registries with proper authentication
   - **Kubernetes**: Configure ImagePullSecrets or use cloud-native authentication methods

5. **Multi-Architecture Support**: Consider building and pushing images for multiple architectures (amd64, arm64) to support different deployment targets.

6. **Image Scanning**: Regularly scan images for vulnerabilities, especially when pulling from public registries or before deploying to production.

7. **Registry Mirroring**: In environments with limited external access (some HPC systems), consider setting up registry mirrors or proxies.

## Registry Authentication Methods

Different registries may require different authentication approaches:

- **Docker Hub**: Requires a CLI access token (generated in account security settings) rather than your account password
- **Quay.io**: Supports both username/password and OAuth tokens
- **NVIDIA NGC**: Requires an API key from your NGC account
- **GitHub Container Registry**: Uses GitHub Personal Access Tokens (PATs)

Always consult the specific registry's documentation for authentication requirements.

## Building from Registry Images

You can use registry images as base layers when building your own containers across all environments:

**Dockerfile:**
```dockerfile
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y python3
COPY app.py /app/
CMD ["python3", "/app/app.py"]
```

**Apptainer Definition File:**
```
Bootstrap: docker
From: docker://ubuntu:22.04

%post
    apt-get update
    apt-get install -y python3

%files
    app.py /app/

%runscript
    python3 /app/app.py
```

**Podman Build:**
```bash
podman build -t myimage:tag -f Dockerfile .
```

This approach allows you to build upon existing images from registries while customizing them for your specific workflows, whether in local development, cloud deployment, HPC environments, or Kubernetes clusters.

## References

- [Apptainer Documentation: Support for Docker / OCI Containers](https://apptainer.org/docs/user/latest/docker_and_oci.html)
- [Open Container Initiative](https://opencontainers.org/)

