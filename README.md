# AZ-204-Certification-notes
# Azure Container Registry (ACR)

## 📌 Overview
- **Managed Docker registry** service (based on Docker Registry 2.0).  
- Stores and manages **container images & artifacts** (Docker, Helm charts, OCI images).  

## 🔄 Key Use Cases
- **Pull images** for deployments in:  
  - **Kubernetes (AKS), DC/OS, Docker Swarm**  
  - **Azure services** (App Service, Batch, Service Fabric).  
- **Push images** from CI/CD pipelines (Azure Pipelines, Jenkins).  

## ⚙️ ACR Tasks (Automated Builds)
- **Automatically rebuild images** when:  
  - Base images are updated.  
  - Code is committed to Git.  
- Supports **multi-step tasks** (build, test, patch in parallel).  

## 💰 Service Tiers
| Tier | Best For | Key Features |  
|------|----------|--------------|  
| **Basic** | Learning/Dev | Same features as higher tiers, but limited storage/throughput |  
| **Standard** | Production | More storage & throughput than Basic |  
| **Premium** | High-volume/Global | Geo-replication, content trust (image signing), private endpoints |  

## 🗂 Supported Artifacts
- **Docker images** (Linux & Windows).  
- **Helm charts** & **OCI-compliant images**.  

## 🔐 Security & Integration
- **Microsoft Entra ID (Azure AD) integration**.  
- **Private endpoints** for restricted access.  

## 🚀 AZ-204 Exam Tips
- **ACR Tasks** is key for **automation scenarios** (Git triggers, base updates).  
- **Tier selection** impacts cost/performance (e.g., geo-replication in Premium).  
- Often integrated with **AKS, Azure Pipelines, and App Service**.  

## 🔒 Security & Storage Features
- **Encryption-at-rest**:  
  - All images/artifacts are automatically encrypted.  
  - Optional: Add customer-managed keys (CMK) for extra encryption.  
- **Geo-redundancy**:  
  - Data stored in the registry's region (+ paired region for most geographies).  
  - **Brazil South/Southeast Asia**: Data confined to single region (compliance).  
- **Geo-replication** (Premium tier only):  
  - Replicates registry across regions for HA and faster access.  
- **Zone redundancy** (Premium tier):  
  - Replicates registry across 3+ availability zones in enabled regions.  

## 📦 Storage & Maintenance
- **Scalable storage**:  
  - Unlimited repositories/images/tags (up to registry storage limit).  
  - ⚠️ Performance impact with excessive tags/repositories → clean up unused artifacts.  
- **Data recovery**:  
  - No automatic recovery during regional outages (use geo-replication for HA).  

## ⚡ ACR Tasks (Automation)
### 🔧 Task Types
| Task Type               | Trigger                                                                 |
|-------------------------|-------------------------------------------------------------------------|
| **Quick Task**          | On-demand build/push (`az acr build`). No local Docker needed.         |
| **Source Code Trigger** | Git commits/PRs (GitHub/Azure DevOps).                                 |
| **Base Image Trigger**  | Automatically rebuilds when base images (e.g., `FROM ubuntu:latest`) update. |
| **Scheduled Task**      | Timer-based builds (e.g., nightly tests).                               |
| **Multi-Step Task**     | YAML-defined workflows (build → test → deploy).                        |

### 🌍 Multi-Platform Support
```yaml
--platform Linux/arm64    # ARM64 Linux
--platform Windows/amd64  # AMD64 Windows
```
- Tier Selection:

  - Premium: Geo-replication, content trust (image signing), private endpoints.

  - Basic/Standard: No cross-region replication.

- Triggers:

  - Know how to set up Git-based vs. base-image triggers.

- Multi-step tasks:

  - YAML-based pipelines for CI/CD (build → test → deploy).

- Security:

  - Encryption is automatic; CMK is optional.

  - Private endpoints restrict access (Premium tier).
