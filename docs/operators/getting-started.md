# Getting Started

Operator deployments are not self-serve. Because Saturn Cloud integrates directly with your bare metal and networking infrastructure, the initial deployment is a joint effort between Saturn Cloud, your team, and your infrastructure providers.

## Step 1: Talk to sales

Start at [saturncloud.io/help](/help) or email [support@saturncloud.io](mailto:support@saturncloud.io). We'll schedule a call to understand your infrastructure, your customers, and what you need the platform to do.

Come to that conversation with:

- How many GPU servers you're starting with, and what GPU models
- Whether you already have a BMaaS provider, or need one
- Whether you have a networking provider, or need one
- Roughly how many tenants you're planning to onboard initially
- Any branding or domain requirements

## Step 2: Infrastructure requirements

After the initial call, Saturn Cloud will work with you to document the specific infrastructure requirements for your deployment. The main things we need to confirm:

**Bare metal provider**: Saturn Cloud integrates with your BMaaS provider's API to orchestrate servers. You can bring your own BMaaS provider or use one of our partners. Either way, the BMaaS provider needs to be in place before Saturn Cloud can deploy.

**Networking**: Some BMaaS providers also handle network fabric. If yours does not, we'll identify a networking provider. Saturn Cloud needs to be able to call the network provider's API to configure tenant network isolation.

**Datacenter access**: Saturn Cloud runs inside your datacenter. We need a small footprint for the Saturn Cloud control plane itself, separate from the GPU servers you're selling.

## Step 3: Joint deployment

Saturn Cloud deploys the platform in coordination with your BMaaS provider. This is not something you run yourself. The deployment involves:

- Installing the Saturn Cloud control plane in your datacenter
- Configuring the BMaaS API integration so Saturn Cloud can provision bare metal, VMs, managed Kubernetes, and managed Slurm on request
- Setting up the networking integration for tenant isolation
- Configuring your domain and branding
- Smoke testing end-to-end provisioning

Expect this phase to take a few days to a few weeks depending on how much infrastructure is already in place.

## Step 4: Go live

Once the platform is deployed and verified, your customers can sign up directly through the portal. There is no per-tenant provisioning step on your end -- the portal is self-service. See [Tenant Management](<docs/operators/tenant-management.md>) for how tenants and capacity work.

## Questions

Contact [support@saturncloud.io](mailto:support@saturncloud.io) for any questions about the process.
