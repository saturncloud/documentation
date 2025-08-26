# Platform, Infrastructure, and DevOps Engineering

## I already have a cloud provider. Do I need Saturn Cloud?

Saturn Cloud is actually a cloud product that runs within your existing infrastructure. AI/ML engineers typically need access to

- Development machines with GPU access
- The ability to deploy LLMs, ML models and dashboards (like Streamlit)
- The ability to schedule jobs
- The ability to run parallel training and run large experiments.

Platform engineers usually start by provisioning these resources for AI/ML engineers manually, and then over time build out their own automation or platforms around them.

**As a platform engineer you have the technical capabilities to build and manage this infrastructure. The real question is whether you want to spend the time building, supporting, maintaining, and being on call for this layer of infrastructure, or if you have better things to work on with your time.**

## Do I need to run everything in Saturn Cloud?

No! Saturn Cloud is typically installed into your existing cloud provider, often within an existing VPC. This means you can build out other infrastructure on your cloud provider directly and still connect to it from workloads running in  Saturn Cloud. In addition, since Saturn Cloud just runs your docker containers with code from your git repositories, it’s easy to migrate workloads into and out of the product.

The rest of these sections address common questions other platform engineers have had about Saturn Cloud. This will help you figure out if it is a good fit for you and your company.

## Where does Saturn Cloud run? What is a GPU Cloud?

Saturn Cloud can currently run in your AWS, GCP, Azure, Oracle and Nebius cloud accounts - often within your existing VPC. Nebius is one of the top tier GPU clouds and we are focused on on-boarding many new GPU clouds in the near future. GPU clouds have exploded in the market in the past few years because the hyperscalers (AWS, GCP, Azure) often lack sufficient on-demand capacity for most customers. This means in order to use top tier GPUs (like H100s and H200s) on AWS you may actually have to commit to paying for them 24/7 for 1-3 years. In addition, the on-demand GPU price on GPU clouds is often 5x cheaper than the major cloud providers. For example an 8xH100 machine on AWS (p5.48xlarge) is $98/hour. That same configuration on Nebius is $23/hr. Leveraging GPU clouds can save you $54,000 per month just on a single 8xH100 machine. GPUs can be hard to get at affordable prices. We focus on helping you tap into global GPU capacity so you can easily access the latest and greatest GPUs.

## Security

All data stored in Saturn Cloud is encrypted at rest (we typically use encrypted block storage on your cloud provider of choice) as well as encrypted in transit (we deploy calico with wireguard within our Kubernetes clusters to encrypt all network traffic and use TLS on all public endpoints).

Whenever possible - we encourage our customers to configure SSO with their existing IDP (usually Okta or Azure AD). We also encourage customers to install Saturn Cloud within their VPN so that it is not exposed to the internet (even though it is perfectly safe to do so).

Saturn Cloud is also compatible with any existing security software you wish to deploy. In the past we have integrated with many different security tools, including (but not limited to)

- Crowdstrike for intrusion detection and response
- Datadog
- Orca Security

<img src="/images/docs/saturn-architecture-onpremise.webp" class="doc-image"/>

## Disaster Recovery

All of our state is typically stored in encrypted block storage. If your cloud provider supports snapshots (most of them do), we maintain snapshots for 30 days. The entire state of the application can be restored from snapshots if necessary. We have also often recovered workspaces or files that have been accidentally deleted by users by recovering from snapshots.

## Automation

While Saturn Cloud does have a nice easy to use point-and-click UI, every action you can take in the UI can be done via the Saturn Cloud API.

### The Saturn Cloud API and Saturn Client

We have [an HTTP API](https://api.saturncloud.io/). There is also a pip installable python library, `saturn-client` that can be imported into Python, and also has a full featured CLI

### Saturn Cloud recipes

Much of Saturn Cloud automation is driven by Saturn Cloud recipes. Recipes are a YAML encoding of a Saturn Cloud resource. Recipes can be exported for every resource, and can also be used to update the same resources.

### Common automation use cases

Here are a few common tasks we’ve seen our customers perform

- Building images in GitHub actions and automatically registering those images with Saturn Cloud
- Automatically updating production Saturn Cloud deployments when production branches are updated via CI/CD
- Automatically create IAM roles and assign them to users when they join the company.

{{% enterprise_docs_view %}}
