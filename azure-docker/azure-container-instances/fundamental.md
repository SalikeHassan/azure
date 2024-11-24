# Running Containers on Azure Container Instances (ACI)

## What is Azure Container Instances (ACI)?
- **Definition**: ACI is the fastest and easiest way to run containers in Azure.
- **Key Features**:
  - **Serverless Platform**: No need to provision container hosting infrastructure.
  - **Pay-Per-Use**: Per-second billing for container runtime, making it cost-effective for short-lived tasks.
  - **Platform-Managed**: Automatically provisions servers to run containers.
  - Supports **Linux** and **Windows** containers (Linux containers are faster to start due to smaller image sizes).

## Use Cases for ACI

### Not Ideal
- Long-running containers (e.g., websites, databases) because the cost can be higher than using Virtual Machines (VMs) for 24/7 workloads.

### Best Fit
1. **Short-lived experiments**:
   - Quickly spin up Elasticsearch or MongoDB for testing.
2. **CI/CD Builds**:
   - Start containers on demand for software builds and avoid idle infrastructure costs.
3. **Batch Jobs**:
   - Containers for scheduled nightly jobs.
4. **Load Testing**:
   - Spin up multiple containers to simulate load.
5. **Elastic Workloads**:
   - Handle traffic spikes by integrating ACI with Kubernetes using the **ACI Connector (Virtual Kubelet)**.

## Key Features of ACI

1. **Automation Options**:
   - Use **Azure CLI**, **PowerShell**, **C# SDK**, or **ARM Templates**.
2. **Networking Features**:
   - Assign **public IP** and set **DNS name prefixes**.
   - Specify **open ports**.
3. **Storage**:
   - Mount Azure File Shares as volumes.
   - Support for secrets and Git repositories as mounted volumes.
4. **Restart Policies**:
   - `always`, `never`, or `onfailure`.
5. **Container Groups**:
   - Group of containers sharing the same server and resources.
   - Similar to **Kubernetes pods**.
   - Enables patterns like **sidecar containers**.
6. **Environment Variables**:
   - Configurable at creation for customization.

## Azure CLI Commands for ACI

### 1. Create a Container Group
```bash
az container create \
    -g <resource-group> \
    -n <container-group-name> \
    --image <image-name> \
    --ip-address public \
    --dns-name-label <dns-prefix> \
    --ports <port-numbers> \
    --cpu <cpu-count> \
    --memory <memory-size>
---
-g: Resource group name.
-n: Container group name.
--image: Docker image name (e.g., from Docker Hub or Azure Container Registry).
--ip-address public: Assigns a public IP.
--dns-name-label: Creates a friendly DNS name (e.g., <prefix>.region.azurecontainer.io).
--ports: Specifies ports to open.
--cpu and --memory: Allocates resources (default: 1 CPU, 1.5 GB memory).
```
### 2. az container logs \
 ```bash  
    -g <resource-group> \
    -n <container-group-name>
```


