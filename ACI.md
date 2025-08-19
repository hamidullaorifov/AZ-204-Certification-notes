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

## 🔄 Restart Policies
| Policy        | Behavior                                                                 | Billing Implications                     |
|---------------|--------------------------------------------------------------------------|------------------------------------------|
| **Always**    | Containers always restart (default).                                     | Billed continuously while running.       |
| **Never**     | Containers run **once** and never restart.                               | Billed only until process completes.     |
| **OnFailure** | Restarts **only if** process fails (non-zero exit code). Runs at least once. | Billed until successful completion.      |

## 🚀 Run-to-Completion Tasks
- **Use Case**: Ideal for one-off tasks (builds, tests, batch jobs).
- **Billing**: Pay **only while the container runs** (seconds granularity).
- **Status**: Shows `Terminated` when completed (`Never`/`OnFailure` policies).

### Example: Run a Task with `OnFailure` Restart
```bash
az container create \
    --resource-group myResourceGroup \
    --name myjob \
    --image mytaskimage \
    --restart-policy OnFailure
```
### 📌 Exam Tips
1. Cost Optimization:

  - Use Never or OnFailure for short-lived tasks to avoid unnecessary charges.

2. Exit Codes:

  - OnFailure triggers restarts only for non-zero exits.

3. Status Checks:

  - Verify task completion via container status (Terminated).

## 💡 Scenario
Problem: Run a CI/CD build job cost-effectively.
Solution:

```bash
az container create \
    --name build-agent \
    --image alpine/git \
    --command "git clone https://github.com/my/repo && ./build.sh" \
    --restart-policy Never  # Stop after build, even if successful
```

# Environment Variables in ACI

## 🌟 Key Concepts
- **Dynamic Configuration**: Pass runtime settings to containers via environment variables.
- **Secure Values**: Safely inject secrets (passwords, keys) without exposing them in container properties.

---

## 🛠 Setting Environment Variables

### CLI Example (Non-Secure)
```bash
az container create \
    --resource-group myResourceGroup \
    --name mycontainer \
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest \
    --environment-variables 'NumWords'='5' 'MinLength'='8' \
    --restart-policy OnFailure
```
### YAML Example (Secure + Non-Secure)
```yaml
apiVersion: 2018-10-01
properties:
  containers:
  - name: mycontainer
    properties:
      environmentVariables:
        - name: 'NOTSECRET'    # Non-secure
          value: 'exposed-value'
        - name: 'SECRET'       # Secure (hidden in logs/UI)
          secureValue: 'my-password'
      image: nginx
  osType: Linux
```
## Security Best Practices
1. Use secureValue for:
  - Passwords, API keys, connection strings.

  - Any sensitive data that shouldn't appear in logs/UIs.

2. Avoid:

  - Hardcoding secrets in container images.

  - Using regular value for sensitive data.

## 📌 Exam Tips
- Syntax Matters:

  - CLI: Use single quotes in Bash ('VAR'='value'), double quotes in CMD ("VAR"="value").

  - YAML: Indentation is critical for environmentVariables.

- Visibility:

  - Secure values are hidden in Azure Portal/CLI outputs.

  - Non-secure values are visible everywhere.

- Common Use Cases:

  - Database connection strings.

  - Feature flags for apps.

  - Credentials for external APIs.

## 💡 Example Scenario
Problem: Deploy a container needing a database password.
Solution:

```bash
az container create \
    --name myapp \
    --image myapp:latest \
    --environment-variables 'DB_HOST'='db.example.com' \
    --secure-environment-variables 'DB_PASSWORD'='s3cr3t!' \
    --restart-policy OnFailure
```

# Mount Azure File Shares in ACI
## 📌 Key Concepts
- **Persistent Storage**: Azure File Shares allow stateful containers by mounting SMB shares.
- **Use Cases**: Logs, config files, or any data needing persistence beyond container lifecycle.
- **Limitations**:
  - Only **Linux** containers (Windows not supported).
  - Container must run as **root**.
  - Supports **CIFS/SMB** protocol only.

---

## 🛠 Mounting a File Share

### CLI Example
```bash
az container create \
    --name hellofiles \
    --image mcr.microsoft.com/azuredocs/aci-hellofiles \
    --dns-name-label aci-demo \
    --ports 80 \
    --azure-file-volume-account-name $STORAGE_ACCOUNT \
    --azure-file-volume-account-key $STORAGE_KEY \
    --azure-file-volume-share-name $SHARE_NAME \
    --azure-file-volume-mount-path /aci/logs/
```
### YAML Example
```yaml
apiVersion: '2019-12-01'
properties:
  containers:
  - name: hellofiles
    properties:
      image: mcr.microsoft.com/azuredocs/aci-hellofiles
      volumeMounts:
      - mountPath: /aci/logs/      # Container path
        name: filesharevolume
  volumes:
  - name: filesharevolume
    azureFile:
      sharename: acishare          # Azure Files share
      storageAccountName: <account>
      storageAccountKey: <key>
```
## 🔄 Mounting Multiple Volumes
### YAML/ARM Template Snippet
```yaml
volumes:
- name: volume1
  azureFile:
    shareName: share1
    storageAccountName: myStorageAccount
    storageAccountKey: <key>
- name: volume2
  azureFile:
    shareName: share2
    storageAccountName: myStorageAccount
    storageAccountKey: <key>

containers:
- name: mycontainer
  properties:
    volumeMounts:
    - name: volume1
      mountPath: /mnt/share1/
    - name: volume2
      mountPath: /mnt/share2/
```
### 📌 Exam Tips
1. Prerequisites:
  - Pre-create Azure File Share and get storage account key.
  - Unique dns-name-label per region.
2. Security:
  - Use storageAccountKey but consider Key Vault for production.
3. Common Errors:
  - 403 Forbidden: Incorrect storage key/share name.
  - Mount denied: Container not running as root.

## 💡 Example Scenario
Problem: Run a web app with persistent logs.
Solution:

1. Create Azure File Share (logs-share).

2. Mount to /var/log/app in container.

3. Logs persist even if container crashes.
