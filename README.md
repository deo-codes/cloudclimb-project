# \#  CloudClimb Project 1 — Secure Azure Application Infrastructure

# 

#### \## Project Overview

# 

#### CloudClimb Project 1 is a hands-on Azure infrastructure project designed to build a secure, segmented cloud environment while developing practical cloud engineering and security skills.

#### 

#### The project is being developed incrementally. Each phase introduces new Azure resources and operational responsibilities while maintaining the network and security foundations established in previous phases.

#### 

#### 

#### 

#### During Week 1, I designed and deployed the core Azure network foundation, including a resource group, virtual network, three workload-specific subnets, Network Security Groups (NSGs), routing, and a consistent naming and tagging strategy.

#### 

#### During Week 2, I expanded the environment by deploying an Ubuntu Linux virtual machine into the Application Subnet and configuring secure SSH administration, network access controls, connectivity validation, troubleshooting, and Azure CLI management.

#### 

#### The long-term goal of the project is to build the infrastructure required to host the Memos application while applying Azure networking, identity, security, monitoring, and operational best practices.

#### 

#### \---

#### 

#### \## Project Architecture

#### 

#### The environment uses a three-tier network design inside an Azure Virtual Network:

#### 

#### \- \*\*Application Tier\*\* — Hosts application compute resources.

#### \- \*\*Data/Backend Tier\*\* — Reserved for backend and data services.

#### \- \*\*Management Tier\*\* — Reserved for administrative and management workloads.

#### 

#### Separating these workloads into dedicated subnets provides clearer security boundaries and allows network access controls to be applied independently to each tier.

#### 

#### \### Azure Environment

#### 

#### | Resource | Configuration |

#### |---|---|

#### | Resource Group | `rg-cloudclimb-dev-eastus` |

#### | Virtual Network | `vnet-cloudclimb-dev-eastus` |

#### | VNet Address Space | `10.10.0.0/16` |

#### | Application Subnet | `snet-app-dev-eastus` — `10.10.1.0/24` |

#### | Data/Backend Subnet | `snet-data-dev-eastus` — `10.10.2.0/24` |

#### | Management Subnet | `snet-mgmt-dev-eastus` — `10.10.3.0/24` |

#### | Application NSG | `nsg-app-dev-eastus` |

#### | Data NSG | `nsg-data-dev-eastus` |

#### | Management NSG | `nsg-mgmt-dev-eastus` |

#### 

#### \---



#### \### Week 1 Architecture

#### 

#### The Week 1 architecture established the network foundation before any application compute resources were deployed.

#### 

#### The architecture consisted of:

#### 

#### \- One Azure Resource Group

#### \- One Azure Virtual Network

#### \- Three segmented subnets

#### \- Three Network Security Groups

#### \- Azure system routing

#### \- Reserved network capacity for future workloads

#### 

#### !\[Week 1 Network Architecture](diagrams/cloudclimb-week01-networkarc-design.png)

#### 

#### \### What I Learned

#### 

#### Week 1 helped me understand why cloud infrastructure should be designed before workloads are deployed.

#### 

#### I learned how CIDR ranges define network address space, how subnets can separate workloads into security boundaries, how NSGs provide network-level traffic controls, and how Azure system routing supports communication within a virtual network.

#### 

#### I also learned the importance of planning IP address ranges so that future resources can be added without creating overlapping address spaces or requiring unnecessary network redesign.

