# CloudClimb Project 1 — Secure Azure Application Infrastructure

## Project Overview

CloudClimb Project 1 is a hands-on Azure infrastructure project designed to build a secure, segmented cloud environment while developing practical cloud engineering and security skills.

The project is being developed incrementally. Each phase introduces new Azure resources and operational responsibilities while maintaining the network and security foundations established in previous phases.

During Week 1, I designed and deployed the core Azure network foundation, including a resource group, virtual network, three workload-specific subnets, Network Security Groups (NSGs), routing, and a consistent naming and tagging strategy.

During Week 2, I expanded the environment by deploying an Ubuntu Linux virtual machine into the Application Subnet and configuring secure SSH administration, network access controls, connectivity validation, troubleshooting, and Azure CLI management.

The long-term goal of the project is to build the infrastructure required to host the Memos application while applying Azure networking, identity, security, monitoring, and operational best practices.

---

## Project Architecture

The environment uses a three-tier network design inside an Azure Virtual Network:

* **Application Tier** — Hosts application compute resources.
* **Data/Backend Tier** — Reserved for backend and data services.
* **Management Tier** — Reserved for administrative and management workloads.

Separating these workloads into dedicated subnets provides clearer security boundaries and allows network access controls to be applied independently to each tier.

### Azure Environment

| Resource | Configuration |

|---|---|

| Resource Group | `rg-cloudclimb-dev-eastus` |

| Virtual Network | `vnet-cloudclimb-dev-eastus` |

| VNet Address Space | `10.10.0.0/16` |

| Application Subnet | `snet-app-dev-eastus` — `10.10.1.0/24` |

| Data/Backend Subnet | `snet-data-dev-eastus` — `10.10.2.0/24` |

| Management Subnet | `snet-mgmt-dev-eastus` — `10.10.3.0/24` |

| Application NSG | `nsg-app-dev-eastus` |

| Data NSG | `nsg-data-dev-eastus` |

| Management NSG | `nsg-mgmt-dev-eastus` |

---

### Week 1 Architecture

The Week 1 architecture established the network foundation before any application compute resources were deployed.

The architecture consisted of:

* One Azure Resource Group
* One Azure Virtual Network
* Three segmented subnets
* Three Network Security Groups
* Azure system routing
* Reserved network capacity for future workloads

![Week 1 Network Architecture](diagrams/cloudclimb-week01-networkarc-design.png)

### What I Learned

Week 1 helped me understand why cloud infrastructure should be designed before workloads are deployed.

I learned how CIDR ranges define network address space, how subnets can separate workloads into security boundaries, how NSGs provide network-level traffic controls, and how Azure system routing supports communication within a virtual network.

I also learned the importance of planning IP address ranges so that future resources can be added without creating overlapping address spaces or requiring unnecessary network redesign.

---

## Week 2 — Application Compute

### Objective

The objective of Week 2 was to introduce the first compute workload into the Azure environment while maintaining the network segmentation and security foundation established during Week 1.

I deployed an Ubuntu Linux virtual machine into the Application subnet and configured secure administrative access using SSH public-key authentication.

This phase also focused on validating network connectivity, managing the VM through Azure CLI, troubleshooting SSH connectivity, and updating the architecture documentation to reflect the new compute layer.

### Linux Virtual Machine

The first compute resource deployed into the environment was:

| Configuration | Value |

|---|---|

| VM Name | `vm-app-dev-eastus` |

| Operating System | Ubuntu Server 22.04 LTS |

| Architecture | ARM64 |

| VM Size | `Standard_B2pts_v2` |

| Subnet | `snet-app-dev-eastus` |

| Private IP | `10.10.1.4` |

| Authentication | SSH public key |

The VM was placed inside `snet-app-dev-eastus` because it represents the application workload for the CloudClimb environment.

Keeping the application workload in its dedicated subnet allows application traffic and security controls to be managed independently from the Data/Backend and Management tiers.

### Network Interface

The VM's network interface connects the virtual machine to the Application subnet.

The network interface was assigned the private IPv4 address:

`10.10.1.4`

This address allows the VM to communicate within the Azure virtual network according to the network and security rules applied to the environment.

### SSH Administration

Administrative access to the Linux VM was configured using SSH public-key authentication.

SSH access uses:

- Protocol: TCP

- Port: `22`

- Username: `azureuser`

- Authentication: SSH public/private key pair

Separate SSH key pairs were configured for my desktop and laptop environments. This allowed both administrative devices to access the VM without sharing the same private SSH key.

Private keys remain stored on their respective administrative devices.

### Network Security Group

SSH access is controlled by:

`nsg-app-dev-eastus`

The custom inbound rule:

`Allow-SSH-MyIP`

allows TCP port `22` only from an approved public source IP.

This prevents SSH from being intentionally exposed to all Internet source addresses and demonstrates the principle of restricting administrative access to known sources.

### Connectivity Validation

After connecting to the VM, I validated the Linux environment and network configuration using commands such as:

`hostname`

`ip addr show eth0`

`uname -a`

`df -h`

`free -h`

The VM reported the expected hostname and private IP address.

Outbound Internet connectivity was also validated using:

`sudo apt update`

The VM successfully contacted Ubuntu package repositories and retrieved current package metadata, confirming that outbound connectivity was functioning.

### SSH Troubleshooting Exercise

During Week 2, I performed a troubleshooting exercise involving the Application NSG.

The approved source IP configured in the `Allow-SSH-MyIP` rule was changed so that it no longer matched the public IP of the administrative workstation.

A new SSH connection attempt resulted in a connection timeout.

I investigated the network path and reviewed the NSG inbound rule. The source IP mismatch was identified as the cause of the connectivity failure.

After restoring the correct source IP in the NSG rule, SSH connectivity was successfully restored.

This exercise helped demonstrate the difference between network connectivity and authentication failures.

A TCP connection timeout can indicate that traffic is being blocked or cannot reach the destination, while an SSH authentication error such as `Permission denied (publickey)` indicates that the SSH service was reached but the provided credentials were not accepted.

### Azure CLI Management

Azure CLI was configured on my administrative workstation to manage Azure resources from PowerShell.

After authenticating to the correct Azure subscription, I used Azure CLI to inspect and manage the virtual machine.

The VM was deallocated using:

`az vm deallocate --resource-group rg-cloudclimb-dev-eastus --name vm-app-dev-eastus`

The VM power state was verified using Azure CLI after the operation.

Deallocating the VM when it is not required is part of the project's cost-management strategy because compute resources should not remain running unnecessarily during a development lab.

### Week 2 Architecture

The architecture diagram was updated to include the new application compute and administrative access path.

The updated architecture includes:

- Admin Workstation

- SSH management over TCP port 22

- Source IP restriction

- Application Network Security Group

- `Allow-SSH-MyIP` rule

- Network interface

- Private IP `10.10.1.4`

- Ubuntu application VM

- Existing Application, Data/Backend, and Management subnets

![CloudClimb Week 2 Application Compute Architecture](diagrams/week02-application-compute-design.png)

### What I Learned

Week 2 helped me understand how compute resources integrate with an existing Azure network architecture.

I learned that deploying a VM involves more than creating a server. The VM depends on networking components such as the subnet, network interface, IP addressing, Network Security Group rules, and routing in order to communicate securely.

I also gained practical experience with SSH public-key authentication and learned why separate administrative devices can use separate SSH key pairs instead of sharing private keys.

The SSH troubleshooting exercise strengthened my understanding of the difference between network connectivity and authentication problems. When SSH timed out, I learned to investigate the network path and NSG source IP before assuming that the SSH key was the problem.

Finally, I practiced managing Azure resources from PowerShell using Azure CLI and learned how VM deallocation can be incorporated into a development workflow to reduce unnecessary compute usage.
