# Architecture|
| Saturn Cloud control plane | Saturn Cloud | AI Studio (JupyterLab, VS Code, SSH), Token Factory (inference endpoints), the operator management interface, authentication, multi-tenancy, and usage metering |
| BMaaS provider | Your infrastructure | Bare metal provisioning, VM orchestration, managed Kubernetes, managed Slurm |
| Network provider | Your infrastructure (may be part of BMaaS) | Tenant network isolation and fabric configuration |
| GPU servers | Your hardware | Where tenant workloads run. Traffic stays on your network |

The control plane calls the BMaaS and network provider APIs to orchestrate resources. Management plane traffic (provisioning, portal, metering) passes through the Saturn Cloud control plane. GPU workload traffic runs directly on your hardware and does not leave your network.
