# High Security Installations

This article describes approaches are most security conscious customers take when installing Saturn Cloud. Security involves making trade offs. The approaches described here will improve your security, but will make Saturn Cloud harder to use. This is an example - we can work with you to determine which of these approaches is right for you. There are also things we can do, depending on your requirements, that are not mentioned here.

## Block ingress from the internet

As mentioned in our article on [security recommendations](/docs#network-security), for the highest levels of security, Saturn Cloud should be installed without any ingress from the internet.

Under this configuration, you create a VPC with public(optional) and private(mandatory) subnets, and send us the VPC and subnet IDs. Saturn Cloud will be installed in that subnet, and all instances and load balancers will hosted on the private subnet. You will need to add the private subnet to your VPN, so that your users will be able to use Saturn Cloud.

## Restrict egress to the internet

In addition to restricting ingress from the internet, you may opt to restrict egress to the internet as well. This is useful to prevent data scientists from downloading packages from un-trusted sources, or accidentally pushing confidential information to GitHub repositories. This is the most restrictive way to restrict egress to the internet. For something less intrusive, consider [using a transparent proxy](#use-a-transparent-proxy)

To do this - all you need to do is construct your VPC such that there is no egress to the internet (no internet gateway). Doing this will mean that data scientists executing `pip install` or `conda install` commands will run into errors.

To make this easier on users, Saturn Cloud has support for a global configuration script that can be run at the start of every container. This can be used to configure `pip/conda/apt/R` to pull packages from preferred locations. Please contact `support@saturncloud.io` to deploy such configurations. In order to do this - you will need to have an on-premise mirror for packages. Some examples are:

- [Anaconda Server](https://www.anaconda.com/products/server)
- [Posit Package Manager](https://packagemanager.rstudio.com/client/#/)
- [JFrog Artifactory](https://jfrog.com/artifactory/)
- [Sonatype Nexus](https://www.sonatype.com/products/nexus-repository)

Of course your VPC must be configured to allow traffic to your on-premise mirror.


### Conda configuration

The following script can be used to inject conda configuration into containers.

```
# blow away all possible locations of .condarc
# since conda will merge them all available configs

rm -f /etc/conda/.condarc
rm -f /etc/conda/condarc
rm -f /var/lib/.condarc
rm -f /var/lib/conda/condarc
rm -rf /etc/conda/.condarc.d
rm -rf /var/lib/.condarc.d

if [ -z "${XDG_CONFIG_HOME}" ]; then
  rm -f $XDG_CONFIG_HOME/conda/.condarc
  rm -f $XDG_CONFIG_HOME/conda/condarc
  rm -rf $XDG_CONFIG_HOME/conda/condarc.d/
fi

rm -f /opt/conda/.condarc
rm -f /opt/conda/condarc
rm -rf /opt/conda/condarc.d

rm -f /srv/conda/.condarc
rm -f /srv/conda/condarc
rm -rf /srv/conda/condarc.d

rm -f /opt/saturncloud/.condarc
rm -f /opt/saturncloud/condarc
rm -rf /opt/saturncloud/condarc.d

mkdir -p /etc/conda
cat << EOT >> /etc/conda/.condarc
channels:
  - ${your-channel-locations}
EOT
```

### Pip configuration

The following script can be used to inject pip configuration into containers.

```
# blow away all possible pip.conf locations
directories=$(echo ${XDG_CONFIG_DIRS} | tr ":" "\n")
for directory in $directories
do
    rm -f ${directory}/pip/pip.conf
done
rm -f /etc/pip/pip.conf

rm -f /opt/conda/pip.conf
rm -f /opt/conda/envs/saturn/pip.conf
rm -f /srv/conda/pip.conf
rm -f /srv/conda/envs/saturn/pip.conf
rm -f /opt/saturncloud/pip.conf
rm -f /opt/saturncloud/envs/saturn/pip.conf

mkdir -p /etc/pip
cat << EOT >> /etc/pip/pip.conf
[global]
index = ${your-pip-server}
index-url = ${your-pip-server}/simple
trusted-host = ${your-pip-host}
EOT
```

### Apt configuration

```
rm -rf /etc/apt/sources.list
rm -rf /etc/apt/sources.list.d/*

UBUNTU_FOCAL_MIRROR="${your-ubuntu-focal-mirror}"
UBUNTU_BIONIC_MIRROR="${your-ubuntu-bionic-mirror}"

# extract os details
OS_ID=$(grep -Po '(?<=^ID=).*' /etc/os-release)
OS_VERSION=$(grep -Po '(?<=^VERSION_CODENAME=).*' /etc/os-release)
OS_VERSION_ID=$(grep -Po '(?<=^VERSION_ID=).*' /etc/os-release | tr -d '"')

# assign appropriate apt repo
if [[ $OS_ID = "ubuntu" ]] && [[ $OS_VERSION = "focal" ]]
then
cat << EOF | sudo tee /etc/apt/sources.list
deb $UBUNTU_FOCAL_MIRROR focal main restricted
deb $UBUNTU_FOCAL_MIRROR focal-updates main restricted
deb $UBUNTU_FOCAL_MIRROR focal universe
deb $UBUNTU_FOCAL_MIRROR focal-updates universe
deb $UBUNTU_FOCAL_MIRROR focal multiverse
deb $UBUNTU_FOCAL_MIRROR focal-updates multiverse
EOF
elif [[ $OS_ID = "ubuntu" ]] && [[ $OS_VERSION = "bionic" ]]
then
cat << EOF | sudo tee /etc/apt/sources.list
deb $UBUNTU_BIONIC_MIRROR bionic main restricted
deb $UBUNTU_BIONIC_MIRROR bionic-updates main restricted
deb $UBUNTU_BIONIC_MIRROR bionic universe
deb $UBUNTU_BIONIC_MIRROR bionic-updates universe
deb $UBUNTU_BIONIC_MIRROR bionic multiverse
deb $UBUNTU_BIONIC_MIRROR bionic-updates multiverse
EOF
fi
```

### R configuration

```
mkdir -p /usr/local/lib/R/etc/

cat << EOT >> /usr/local/lib/R/etc/Rprofile.site
options(repos = c(CRAN = '${your-cran-mirror}'))
EOT
```

## Use a transparent proxy

Using a transparent proxy is a good way to restrict and audit egress, without forcing data scientists to only consume curated lists of packages. In order to do this - you must have a transparent proxy installed, in a location that can be accessed from the VPC. Then the relevant certificate authority can be added to the containers using the global configuration script.

```
# download the .crt file
sudo wget -O /usr/local/share/ca-certificates/example.crt ${url-to-crt-file}
sudo update-ca-certificates
```

## Use fine grained permissions for Saturn Cloud install and upgrades

Our default permission set that we use for installing and keeping Saturn Cloud is quite broad. A restricted permission set (which is still under active development) is available here. `https://s3.us-east-2.amazonaws.com/saturn-cf-templates/iam-role-restricted.cft`.

The policies in that cloud formation template makes heavy use of tags. You will need to fill in the following values before applying it

- `${saturn-cluster-name}`
- `${saturn-bucket-name}`
- `${external-id}`

In addition, one of the roles uses the OIDC provider for the EKS cluster that will be created in it's trust relationship. Those values will not be available until after the cluster is created - so that role will need to be updated during the installation process.

## Use fine grained permission for Saturn Cloud support

Installation and updates happen about once per quarter. To reduce access to that permission set - you can ask us to use a much smaller set of permissions to provide support and maintenance. The relevant policy is here: `https://s3.us-east-2.amazonaws.com/saturn-cf-templates/iam-self-managed-admin.cft`.  That policy has access to diagnostic information, as well as access to the EKS cluster.

## Restrict access to management and support roles based on time.

Saturn cloud support uses AWS IAM trust relationships in order to assume the necessary roles to maintain Saturn Cloud. Those trust relationships can be disabled when updates or support are not in progress.

## Use a sandboxed AWS account.

We often advise against installing Saturn Cloud in sandboxed AWS accounts during trials, because some customers who do so don't do the necessary steps to connect the sandboxed account with their data. Spoiler alert: a data science platform isn't very useful if it cannot access your data!.

A Sandboxed account is a perfectly reasonable strategy for a Saturn Cloud installation, provided you take the necessary steps to make sure Saturn Cloud can interact with your data. Most data science teams store data in object stores like S3, or in data warehouses (such as Snowflake or Redshift). Enabling cross account access is very easy for all of these tools.

The biggest issue is typically access to APIs that are deployed in private VPCs. It is a good idea to setup the VPC for Saturn Cloud using CIDR blocks compatible with the rest of your infrastructure so that you will be able to peer the VPCs together in the future. Using this approach, you can trial Saturn Cloud in a sandbox account, and turn that account into the production Saturn Cloud account after you decide to use Saturn Cloud longer term.

## Use SSO

As mentioned in our article on [security recommendations](/docs#sso-and-authentication), We definitely recommend integrating Saturn Cloud with the identity provider for your company.

At Saturn Cloud, SSO is available to advanced tier subscribers via [Auth0](https://auth0.com/). Auth0 can connect to virtually any identity provider(IDP). We have documentation for [Okta](/docs), [Azure AD](/docs), and [Google Sign On](/docs), but almost any identity provider can be supported.

SSO is used for authentication, but you can also use it for authorization (determining who should have a Saturn Cloud account). You can:
- Authorize all users in you IDP to have access to Saturn Cloud.
- Authorize all users with a specific email domain to have access to Saturn Cloud.
- Authorize all users within specific groups in your IDP to have access to Saturn Cloud.


## Use IAM Roles instead of IAM users


As mentioned in our article on [security recommendations](/docs#iam-roles), We recommend using IAM roles over IAM users. At Saturn Cloud, we leverage [IAM roles for service accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html). This enables us to map IAM roles to [Saturn Cloud users, groups and containers](/docs).
{{% enterprise_docs_view %}}
{{% security_docs_view %}}
