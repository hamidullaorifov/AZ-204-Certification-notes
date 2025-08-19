# Azure Container Apps - AZ-204 Exam Notes

## 🚀 Overview
- **Serverless containers** on Azure Kubernetes Service (AKS).
- **Use cases**: APIs, microservices, event-driven apps, background jobs.
- **Key features**:
  - Autoscaling (HTTP, CPU/memory, KEDA triggers).
  - Multiple revisions & traffic splitting (A/B testing, blue/green).
  - Built-in HTTPS ingress & service discovery.
  - Dapr integration for microservices.

---

## 🌐 Core Concepts
### 1. **Environments**
- Logical boundary for container apps (shared VNet, Log Analytics).
- **Same environment** for:
  - Microservices communication (Dapr).
  - Shared logging/monitoring.
- **Separate environments** for:
  - Isolation (compute/resources).
  - Security boundaries.

### 2. **Scaling**
| Trigger               | Example                          | Scale to Zero? |
|-----------------------|----------------------------------|----------------|
| HTTP traffic          | API requests                     | ✅ Yes         |
| CPU/Memory            | Batch processing                 | ❌ No          |
| KEDA (custom)         | Queue length (Azure Storage)     | ✅ Yes         |

### 3. **Authentication**
- **Built-in auth providers**:
  - Microsoft Identity, Google, GitHub, etc.
  - No code required (`/.auth/login/<provider>`).
- **Modes**:
  - `Require authentication`: Restrict access.
  - `Allow unauthenticated`: Optional auth.

---

## 🛠 Configuration Examples
### ARM Template (Single Container)
```json
"containers": [{
  "name": "main",
  "image": "myapp:latest",
  "env": [
    { "name": "PORT", "value": "80" },
    { "name": "DB_PWD", "secretRef": "db-secret" }
  ],
  "resources": { "cpu": 0.5, "memory": "1Gi" }
}]
```

## 🔐 Security & Auth Flow

```flowchart LR
  A[Request] --> B[Auth Sidecar]
  B -->|Valid Token| C[App Container]
  B -->|Invalid| D[Redirect to Login]
```
- Headers injected:

  - X-MS-CLIENT-PRINCIPAL: User identity.

  - X-MS-TOKEN-<PROVIDER>: Provider token.

## 📌 Exam Tips
1. Scaling:

    - KEDA supports custom scalers (e.g., Cosmos DB, Kafka).

    - HTTP scaling is most common.

2. Dapr:

    - Use for service invocation, pub/sub, state management.

3. Revisions:

    - Traffic splitting enables canary deployments.

4. Limitations:

    - No Windows containers.

    - No privileged containers (root access).

## 💡 Example Scenario
Problem: Deploy a microservice with auth and autoscaling.
Solution:

1. Create environment with VNet.

2. Deploy container app:

3. Enable Microsoft Identity auth.

4. Set HTTP scaling rules.

5. Add Dapr for service discovery.

# Azure Container Apps: Revisions & Secrets - AZ-204 Notes

## 🔄 Revisions Management
- **What**: Immutable snapshots of app versions.
- **Created when**: Revision-scope changes occur (e.g., image update, env vars).
- **Key commands**:
  ```bash
  # Update app (creates new revision if needed)
  az containerapp update --name myapp --image myapp:v2

  # List revisions
  az containerapp revision list --name myapp -o table
  ```
- ** Traffic splitting:** Route % of traffic between revisions (A/B testing).

### 🔐 Secrets Management
 - Scoped to app (not revisions). Changes don’t auto-create revisions.
 -  Define secrets:
    ```bash
    az containerapp create \
      --name myapp \
      --secrets "db-conn=$CONN_STRING" "api-key=12345"
    ```
  - Reference secrets in env vars:

    ```bash
    --env-vars "DB_CONN=secretref:db-conn"
    ```
  - Key points:

      - No direct Key Vault integration → Use Managed Identity + SDK.

      - Update secrets? Deploy new revision or restart existing one.

## ⚡ Dapr Integration
### Core APIs
| API	| Use Case  |
| Service Invocation |	Secure microservices communication |
| State Management |	CRUD operations with state stores |
| Pub/Sub |	Event-driven messaging |
| Actors	| Scalable units of work
### Enable Dapr (ARM Snippet)
```json
"dapr": {
  "enabled": true,
  "appId": "cart-service",
  "appPort": 3000
}
```
 - Sidecar: Runs on HTTP port 3500/gRPC 50001.

 - Components: Shared across apps (scope with appIds).

## 📌 Exam Tips
1. Revisions:

    - Use --revision-suffix for custom names.

    - Revert by activating old revision.

2. Secrets:

    - Always use secretref: in env vars.

    - Rotate secrets via new revisions.

3. Dapr:

    - Know port numbers (3500, 50001).

    - appId must be unique per app.

```text

### Key Takeaways:
- **Revisions** = Version control for apps.
- **Secrets** = Safe env vars (no KV integration).
- **Dapr** = Microservices glue (pub/sub, state, actors).  
```
