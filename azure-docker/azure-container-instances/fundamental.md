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

## Create ACI using terraform
```hcl
FileName: variables.tf

# Define all configurable variables for reusability and clarity

# Resource group name
variable "resource_group_name" {
  description = "The name of the resource group to create."
  default     = "AciResourceGroup"
}

# Azure region
variable "location" {
  description = "The Azure region where resources will be created."
  default     = "West Europe"
}

# Storage account name
variable "storage_account_name" {
  description = "Unique name for the storage account."
  default     = "acicontainerstore"
}

# File share name
variable "file_share_name" {
  description = "The name of the Azure file share."
  default     = "acishare"
}

# Container group name
variable "container_group_name" {
  description = "The name of the Azure container group."
  default     = "aci-sample-group"
}

# Container image
variable "container_image" {
  description = "The Docker image to run in the container."
  default     = "nginx:latest"
}
```
```hcl
FileName: local.tf
# Define local values for derived variables or reusable values
locals {
  container_cpu     = "1"         # CPU allocated to the container
  container_memory  = "1.5"       # Memory allocated to the container
  container_port    = 80          # Port exposed by the container
  storage_tier      = "Standard"  # Storage account tier
  replication_type  = "LRS"       # Replication type for the storage account
  mount_path        = "/mnt/share" # Mount path for the Azure file share in the container
}
```

```hcl
FileName: main.tf
# Define the Azure provider
provider "azurerm" {
  features {}
}

# Create a resource group
resource "azurerm_resource_group" "aci_group" {
  name     = var.resource_group_name
  location = var.location

  # Inline comment: This resource group will house all the ACI resources.
}

# Create a storage account
resource "azurerm_storage_account" "aci_storage" {
  name                     = var.storage_account_name
  resource_group_name      = azurerm_resource_group.aci_group.name
  location                 = azurerm_resource_group.aci_group.location
  account_tier             = local.storage_tier
  account_replication_type = local.replication_type

  # Inline comment: This storage account is used to create and manage a file share for the container.
}

# Create a file share in the storage account
resource "azurerm_storage_share" "aci_share" {
  name                 = var.file_share_name
  storage_account_name = azurerm_storage_account.aci_storage.name

  # Inline comment: File share to be mounted inside the container for persistent storage.
}

# Create the Azure container group
resource "azurerm_container_group" "aci" {
  name                = var.container_group_name
  location            = azurerm_resource_group.aci_group.location
  resource_group_name = azurerm_resource_group.aci_group.name
  os_type             = "Linux"  # Specify the OS type for the container

  # Container definition
  container {
    name   = "sample-container"
    image  = var.container_image  # Docker image for the container
    cpu    = local.container_cpu
    memory = local.container_memory

    # Define the port to expose
    ports {
      port     = local.container_port
      protocol = "TCP"
    }

    # Mount Azure file share as a volume
    volume {
      name                 = "volume1"
      mount_path           = local.mount_path
      storage_account_name = azurerm_storage_account.aci_storage.name
      storage_account_key  = azurerm_storage_account.aci_storage.primary_access_key
      share_name           = azurerm_storage_share.aci_share.name
    }
  }

  # Expose the container with a DNS name
  ip_address_type = "Public"  # Publicly accessible IP
  dns_name_label  = "aci-public-dns-${var.container_group_name}" # DNS label for the container group

  # Inline comment: DNS name will be publicly available as "<dns_name_label>.region.azurecontainer.io".
}
```

```hcl
FileName: output.tf
# Output the DNS name assigned to the container
output "container_dns_name" {
  description = "Publicly accessible DNS name for the Azure Container Instance"
  value       = "${azurerm_container_group.aci.dns_name_label}.${azurerm_resource_group.aci_group.location}.azurecontainer.io"

  # Inline comment: This output provides the complete URL to access the container.
}
```

### Terraform Command
```bash
terraform init
terraform plan -out=tfplan
terraform apply tfplan
terraform destroy
```

## Using Local Docker Images with Azure Container Instances (ACI)
### Overview
This section covers the process of building a Docker image locally, pushing it to Azure Container Registry (ACR), and deploying it to Azure Container Instances (ACI) using Terraform.

### Steps

#### 1. Build and Tag the Docker Image Locally
```bash
dotnet new blazorserver -n MyBlazorApp
cd MyBlazorApp
dotnet run 
```
Create Dockerfile in project root folder
```dockerfile
  # Stage 1: Build the app
  FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
  WORKDIR /app
  COPY *.csproj .
  RUN dotnet restore
  COPY . .
  RUN dotnet publish -c Release -o /publish
  
  # Stage 2: Runtime image
  FROM mcr.microsoft.com/dotnet/aspnet:9.0
  WORKDIR /app
  COPY --from=build /publish .
  EXPOSE 5000
  ENTRYPOINT ["dotnet", "MyBlazorApp.dll"]
```

```bash
docker build -t myacrregistry.azurecr.io/myblazorapp:v1 .
```

#### 2. Push the Image to Azure Container Registry (ACR)
```bash
  az acr login --name myacrregistry
```
```bash
docker push myacrregistry.azurecr.io/myblazorapp:v1
```
Verify image successfully pushed
```bash
az acr repository list --name myacrregistry --output table
```

#### 3. Deploy the Image to Azure Container Instances (ACI) Using Terraform
```hcl
FileName: variables.tf

 variable "acr_name" {
  description = "The name of the Azure Container Registry."
  default     = "myacrregistry"
}

variable "docker_image_name" {
  description = "The name of the Docker image for the Blazor app."
  default     = "myblazorapp"
}

variable "docker_image_tag" {
  description = "The tag for the Docker image."
  default     = "v1"
}
```

```hcl
FileName: main.tf
# Reference the Azure Container Registry
resource "azurerm_container_registry" "acr" {
  name                = var.acr_name
  resource_group_name = azurerm_resource_group.aci_group.name
  location            = azurerm_resource_group.aci_group.location
}

# Use the image from ACR in the container group
resource "azurerm_container_group" "aci" {
  name                = var.container_group_name
  location            = azurerm_resource_group.aci_group.location
  resource_group_name = azurerm_resource_group.aci_group.name
  os_type             = "Linux"

  container {
    name   = "blazor-app-container"
    image  = "${azurerm_container_registry.acr.login_server}/${var.docker_image_name}:${var.docker_image_tag}"
    cpu    = local.container_cpu
    memory = local.container_memory

    ports {
      port     = local.container_port
      protocol = "TCP"
    }
  }

  ip_address_type = "Public"
  dns_name_label  = "blazor-${var.container_group_name}"
}
```

```hcl
FileName: output.tf

output "container_dns_name" {
  description = "Publicly accessible DNS name for the Blazor app container"
  value       = "${azurerm_container_group.aci.dns_name_label}.${azurerm_resource_group.aci_group.location}.azurecontainer.io"
}
```

### Steps Summary
- Use Docker CLI for local image creation.
- Use Azure CLI for ACR authentication and image push.
- Use Terraform for provisioning the container instance and referencing the ACR image.





