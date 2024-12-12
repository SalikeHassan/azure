| **Name of the Subscription**    | **Costing Details**                  | **Usage**                                                         | **Compute**        | **Networking**          | **Key Features**                                               |
|---------------------------------|---------------------------------------|-------------------------------------------------------------------|--------------------|-------------------------|----------------------------------------------------------------|
| **Multi-Tenant**                | Pay-per-operation                    | Best for starting with small workflows and cost optimization.     | Shared             | Public cloud            | Fully managed, low-cost entry point.                          |
| **Workflow Service Plan**       | Per workflow service plan instance   | Ideal for dedicated workloads requiring high scalability.          | Dedicated          | VNET Integration        | Single-tenant runtime with in-app connectors and scaling.     |
| **App Service Environment V3**  | Per App Service Environment instance | Suitable for isolated environments and full app service scalability. | Dedicated          | VNET Integration        | Full isolation with support for large-scale environments.     |
| **Hybrid (Preview)**            | Per vCPU hour of Kubernetes cluster  | Ideal for multi-cloud, on-premises, or Kubernetes-based workflows. | Customer Managed   | Local network access     | Kubernetes Event-Driven Autoscaling (KEDA) for high flexibility. |

- Multi-Tenant: This is the most cost-effective option and works well for lightweight workflows, where operations are charged individually.
- Workflow Service Plan: Offers dedicated compute for larger workflows with VNET integration for security.
- App Service Environment V3: Provides full isolation and is suitable for compliance-heavy workloads or large enterprises.
- Hybrid: Ideal for organizations that want flexibility across hybrid and multi-cloud environments.
