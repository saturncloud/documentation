# Saturn Cloud on Crusoe

Saturn Cloud Enterprise for Crusoe provides a fully managed data science platform deployed directly into your Crusoe Cloud environment. Saturn Cloud Enterprise has a standard configuration for Crusoe Cloud. In a typical installation, we provision a [Crusoe Managed Kubernetes (CMK)](https://docs.crusoecloud.com/orchestration/cmk) cluster in your Crusoe project, configure CPU and GPU node pools optimized for AI workloads, and integrate with Crusoe storage and networking services. Saturn Cloud is hosted publicly with secure authentication by default, with private networking options available if required.

This standard installation creates a ready-to-use platform for machine learning and AI workloads on sustainable, cost-effective GPU infrastructure. Beyond the default setup, Saturn Cloud can customize node pool sizes, GPU configurations, and network settings to meet your organization’s needs.

## Architecture Overview

Saturn Cloud on Crusoe leverages Crusoe Managed Kubernetes (CMK) to provide a scalable, secure, and environmentally sustainable platform for data science teams.

<img src="/images/docs/crusoe-architecture-diagram.webp" class="doc-image"/>

The infrastructure includes:

### Core Components

- **CMK Cluster**: A fully managed Kubernetes cluster with automated control plane management
- **Sustainable Infrastructure**: GPU compute powered by stranded energy and renewable sources
- **Node Pools**: Auto-scaling node groups optimized for different workload types, including A100 GPU nodes
- **Storage**: Integration with Crusoe CSI for persistent data storage
- **Container Registry**: Seamless access to container images and registries

### Network Architecture

Saturn Cloud integrates with Crusoe's high-performance networking infrastructure:

#### 1. Network Integration

- **Dedicated Networking**: High-bandwidth, low-latency connections optimized for AI workloads
- **Location-Based Deployment**: Deployed in your specified Crusoe location (us-east1-a, us-west1-a, etc.)
- **Managed Control Plane**: Crusoe handles all networking for the Kubernetes control plane
- **SSH Access**: Optional SSH access to nodes for debugging and management

#### 2. Network Security

The Crusoe deployment ensures:

- Secure API endpoints with TLS encryption
- Network isolation between node pools and workloads
- Kubernetes-native security policies
- Optional VPC integration for enterprise networking requirements

### Node Pool Architecture

The installation creates specialized node pools optimized for AI and data science workloads.

{{% alert title="Note" color="info" %}}
The node pool configurations below are optimized for Crusoe's AI infrastructure. Saturn Cloud can customize these configurations to match your specific requirements, including:
- Different CPU and GPU instance types
- Custom memory and storage configurations
- Specific location preferences
- Tailored autoscaling policies
- Mixed workload optimizations

Contact support@saturncloud.io to discuss customization options for your deployment.
{{% /alert %}}

#### 1. System Node Pool

- **Purpose**: Runs Saturn Cloud control plane and system services
- **Instance Type**: c1a.4x (16 vCPUs, 32GB RAM)
- **Scaling**: Fixed 1-2 nodes for system stability
- **Storage**: High-performance local storage

#### 2. CPU Node Pools

Saturn Cloud provisions CPU-optimized node pools for various computational requirements:

**Compute-Optimized (c1a.4x)**
- **Configuration**: 16 vCPUs, 32GB RAM
- **Use Case**: General purpose workloads, data preprocessing, development environments
- **Auto-scaling**: 0-100 nodes based on demand
- **Cost**: Optimized pricing for sustained workloads

#### 3. GPU Node Pools

Saturn Cloud provides access to Crusoe's full GPU fleet, including NVIDIA A100, H100, and L40 GPUs. Our standard configuration showcases A100 80GB configurations, though Saturn Cloud can provision any GPU type available in Crusoe's infrastructure to meet your specific requirements.

**Single A100 80GB (a100-80gb.1x)**

- **Configuration**: 1x A100 80GB GPU, 16 vCPUs, 120GB RAM
- **GPU Memory**: 80GB HBM2e per GPU
- **Use Case**: Model development, inference, single-GPU training
- **Auto-scaling**: 0-100 nodes
- **Sustainability**: Powered by stranded energy sources

**Dual A100 80GB (a100-80gb.2x)**

- **Configuration**: 2x A100 80GB GPUs, 32 vCPUs, 240GB RAM
- **GPU Memory**: 160GB total HBM2e
- **Use Case**: Multi-GPU training, larger model development
- **Auto-scaling**: 0-50 nodes

**Quad A100 80GB (a100-80gb.4x)**

- **Configuration**: 4x A100 80GB GPUs, 64 vCPUs, 480GB RAM
- **GPU Memory**: 320GB total HBM2e
- **Use Case**: Large-scale model training, distributed computing
- **Auto-scaling**: 0-25 nodes

**Octa A100 80GB (a100-80gb.8x)**

- **Configuration**: 8x A100 80GB GPUs, 128 vCPUs, 960GB RAM
- **GPU Memory**: 640GB total HBM2e
- **Use Case**: Enterprise-scale training, LLM development, research workloads
- **Auto-scaling**: 0-10 nodes

{{% alert title="GPU Fleet Access" color="info" %}}
The A100 configurations shown above represent our reference implementation. Crusoe offers additional GPU types including H100 and L40 GPUs. Contact support@saturncloud.io to discuss accessing Crusoe's complete GPU portfolio for your specific workload requirements.
{{% /alert %}}

{{% alert title="Sustainable AI Computing" color="success" %}}
Crusoe's GPU infrastructure delivers exceptional sustainability benefits:
- **90% lower carbon footprint** compared to traditional cloud providers
- **Stranded energy utilization** - computing powered by otherwise wasted energy
- **Renewable energy integration** - supporting clean energy grid stability
- **Cost efficiency** - up to 50% lower costs than hyperscaler GPU offerings
{{% /alert %}}

### Security Features

Saturn Cloud on Crusoe is deployed with multiple security controls to protect access, data, and workloads:

- **Managed Authentication**: Secure API access through Crusoe's authentication system
- **SSH Key Management**: Optional SSH access with your public keys
- **Network Isolation**: Kubernetes-native network policies and security groups
- **Encrypted Storage**: All persistent storage encrypted at rest
- **Secure Endpoints**: TLS-encrypted communication for all services
- **RBAC Integration**: Role-based access control for Kubernetes resources

### Storage and Data Access

Saturn Cloud leverages Crusoe storage to ensure reliable data access and high availability:

- **Crusoe CSI**: High-performance Container Storage Interface for persistent volumes
- **Local Storage**: Fast local NVMe storage for temporary workloads
- **External Integration**: Support for external storage services and data sources
- **Snapshot Support**: Volume snapshots for backup and recovery

### Monitoring and Operations

Operational features in Saturn Cloud automate scaling, resource allocation, and system health management:

- **Cluster Autoscaler**: Automatic node provisioning based on workload demands
- **GPU Operator**: NVIDIA GPU resource management and scheduling
- **Network Operator**: Optimized networking for GPU workloads
- **Health Monitoring**: Continuous monitoring of node and cluster health
- **Performance Insights**: Built-in monitoring and performance analytics

### Getting Started

Ready to revolutionize your AI infrastructure with sustainable computing? [contact our team](mailto:support@saturncloud.io) to design your custom Crusoe deployment.

**Next Steps:**

1. Sustainability Assessment: Evaluate your current carbon footprint and cost savings potential
2. Architecture Design: Create an optimized deployment for your AI workloads
3. Pilot Deployment: Start with a proof-of-concept in your Crusoe environment
4. Production Scale: Expand to full production with ongoing optimization support

{{% alert title="Limited Sustainable Capacity" color="warning" %}}
Sustainable GPU capacity is in high demand as organizations prioritize ESG compliance. Contact support@saturncloud.io today to secure your allocation and begin your transition to carbon-neutral AI computing.
{{% /alert %}}

## Installation Process

Saturn Cloud Enterprise for Crusoe is fully managed. The installation steps are:

### Prerequisites

1. **Crusoe Cloud Account**: Sign up at https://console.crusoecloud.com/
2. **API Credentials**: Create API keys from the Crusoe console security section
3. **Location Selection**: Choose your preferred Crusoe location for deployment

### Installation Steps

1. **Prepare Crusoe Credentials**: Set up API credentials in your environment:
   ```bash
   # Add to ~/.crusoe/config
   [default]
   access_key_id="YOUR_ACCESS_KEY"
   secret_key="YOUR_SECRET_KEY"
   ```

2. **Configure Installation Parameters**: Provide the following information to Saturn Cloud support:
   ```bash
   # Crusoe project configuration
   project_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
   cluster_name = "saturn-cluster-crusoe"
   location = "us-east1-a"
   kubernetes_version = "1.30.8-cmk.28"
   ```

3. **Grant Installation Access**: Provide Saturn Cloud support with necessary API permissions

4. **Infrastructure Deployment**: Our team handles the CMK cluster deployment and configuration

5. **Access Your Platform**: Receive your Saturn Cloud URL and admin credentials

## Ongoing Management

Saturn Cloud Enterprise on Crusoe is fully managed, meaning:

- Automatic Kubernetes and GPU driver updates
- 24/7 monitoring and support for sustainable AI workloads
- Performance optimization and cost monitoring
- Sustainability reporting and carbon footprint tracking
- Backup and disaster recovery
- Continuous optimization for cost and environmental impact

## Regional Availability

Saturn Cloud on Crusoe can be deployed in locations where Crusoe Cloud operates:

- us-east1-a (Virginia) - Primary region with full A100 availability
- us-west1-a (California) - West coast deployment option
- Additional locations as Crusoe expands sustainable infrastructure

## Cost and Sustainability Optimization

The Crusoe infrastructure provides unique optimization advantages:

- **Auto-scaling to zero** - minimize costs and environmental impact when idle
- **Stranded energy utilization** - lowest possible carbon footprint
- **Efficient resource allocation** - right-sized instances for different workloads
- **Sustainability reporting** - detailed carbon footprint tracking
- **Cost transparency** - clear, predictable pricing with no hidden fees
- **Green computing incentives** - potential tax benefits for sustainable computing

## Performance and Environmental Benefits

Saturn Cloud on Crusoe delivers unique advantages:

- **Carbon-neutral AI computing** - industry-leading sustainability
- **Cost-effective GPU access** - premium A100, H100, and L40 GPUs at accessible prices  
- **High-performance networking** - optimized for AI workloads
- **Renewable energy integration** - supporting clean energy infrastructure
- **Enterprise reliability** - 99.9% uptime with sustainable operations

{{% enterprise_docs_view %}}
