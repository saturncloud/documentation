# Tenant Management

The Saturn Cloud control plane is a single self-service portal shared across all your tenants. Your customers sign up directly -- there is no per-tenant portal to provision or spin up before they can onboard. As an operator, you have a cross-tenant management view on top of the same portal.

## Tenant isolation

Tenants share the same portal but are fully isolated from each other:

- Each tenant sees only their own resources, users, and usage
- Dedicated compute resources (tenants do not share GPU servers unless you configure shared pools)
- Isolated network segments so tenants cannot reach each other's workloads
- Separate usage and billing records

## How tenants sign up

Customers sign up through the portal on your custom domain and can pay by credit card immediately, with no manual step required on your end. GPU-hours are billed as they are consumed.

This is also compatible with reserved capacity commitments. Customers who have pre-purchased capacity can use that allocation, with on-demand consumption on top if needed.

Quotas are automatically applied at signup to prevent runaway spend. You can adjust quota limits per tenant from the operator interface.

## What tenants can do

Once onboarded, tenants are self-service. They can:

- Provision Kubernetes clusters, Slurm clusters, VMs, and bare metal from their allocated pool
- Launch AI Studio workspaces (JupyterLab, VS Code, SSH)
- Submit jobs and manage deployments via Token Factory
- Invite and manage users within their organization
- View their own usage and resource inventory

Tenants cannot provision beyond the capacity or limits you set, and cannot see or interact with other tenants.

## Usage and billing

Saturn Cloud tracks GPU-hours and other resource usage per user, project, and tenant. From the operator interface you can:

- View usage across all tenants
- Export usage records for billing
- View idle resources (GPU servers that are allocated but not running workloads)

Usage data is available in the operator interface and can be exported. Saturn Cloud does not handle invoicing or payment -- that stays with your billing system. The usage records are the input to whatever billing process you run.

## Managing capacity

You can adjust a tenant's allocated capacity at any time from the operator interface. Adding capacity takes effect immediately. Reducing capacity does not forcibly terminate running workloads -- existing resources continue until the tenant deprovisions them.

If you need to reclaim capacity from a tenant (for example, a non-renewing contract), contact [support@saturncloud.io](mailto:support@saturncloud.io) to coordinate.
