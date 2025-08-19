# Azure Container Instances (ACI) - AZ-204 Exam Notes

## 🚀 Overview
- **Serverless containers**: Run isolated containers without managing VMs or orchestrators.
- **Use cases**: Quick tasks, batch jobs, dev/test, microservices.
- **Alternate to AKS**: Use when full orchestration (scaling, service discovery) isn’t needed.

---

## 🔑 Key Features
| Feature                | Description                                                                 |
|------------------------|-----------------------------------------------------------------------------|
| **Fast startup**       | Containers start in seconds (no VM provisioning).                           |
| **Public access**      | Exposed via IP + FQDN (e.g., `myapp.eastus.azurecontainer.io`).             |
| **Security**           | Hypervisor-level isolation (like VMs).                                      |
| **Custom sizing**      | Specify exact CPU/memory (e.g., 1.5 CPU cores, 2GB RAM).                    |
| **Persistent storage** | Mount Azure Files shares for stateful data.                                 |
| **OS support**         | Linux + Windows containers (same API).                                      |

---

## 📦 Container Groups
- **Definition**: A group of containers sharing a host, lifecycle, network, and storage (similar to Kubernetes pods).
- **Limitations**:
  - **Multi-container**: Only Linux (Windows supports single-container groups).
  - **Networking**: Shared IP + port namespace (no port mapping).

### Example YAML Deployment
```yaml
apiVersion: 2019-12-01
location: eastus
name: my-app
properties:
  containers:
  - name: web
    properties:
      image: nginx
      ports:
        - port: 80
  - name: logger
    properties:
      image: alpine/curl
      command: ["sh", "-c", "while true; do curl http://localhost:80; sleep 10; done"]
  osType: Linux
  ipAddress:
    type: Public
    ports:
      - port: 80
  volumes:
    - name: logs

```
## 🛠 Deployment Options
| Method	| When to Use
|---------|-------------------------------------------------------------------------------|
| YAML	  | Simple deployments (only containers). Concise syntax.                         |
| ARM Template |	Complex deployments (e.g., containers + Azure Files + VNet integration).|

## 🌐 Networking & Storage
- Networking:

  - Containers in a group communicate via localhost:<port>.

  - External access requires exposing ports on the group’s IP.

- Storage:

 - Supported volumes:

   - Azure Files (persistent).

  - Secrets (e.g., env vars).

  - EmptyDir (ephemeral).

  - Git repo clones (CI/CD scenarios).

## 📌 Exam Tips
1. Multi-container groups:

  - Only Linux containers.

  - Use for sidecar patterns (e.g., logging + app containers).

2. ACI vs AKS:

  - ACI: Quick, single-task containers.

  - AKS: Full orchestration (scaling, upgrades).

3. Scenarios:

  - Web app + background worker.

  - Logging/monitoring sidecars.

  - CI/CD build agents (ephemeral).

## 💡 Example Scenario
Problem: Run a web app with a logging sidecar.
Solution:

1. Create a container group with:

  - Web container: NGINX (port 80).

  - Logger container: Curl polling localhost:80.

2. Mount Azure Files for log persistence.

3. Expose port 80 publicly.
      azureFile:
        shareName: logs
        storageAccountName: mystorage
