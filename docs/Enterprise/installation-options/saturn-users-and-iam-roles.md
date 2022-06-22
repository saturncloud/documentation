# IAM Roles for Users and Groups

Saturn Cloud allows administrators to grant access to IAM roles to Saturn users. When a user has an IAM, their resources will be run as that IAM role, granting their resources the permission to connect to other AWS components that the role has permissions for. This can be very useful in limiting who can access items like S3 buckets.

## Requirements

The Saturn Cloud infrastructure is built on top of AWS EKS. Associating IAM roles with Saturn users leverages EKS mechanisms for mapping [IAM roles to pods via service accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html). In order to do this, you must contact support@saturncloud.io in order to enable IRSA for your EKS cluster.

## Limitations

Currently a user or group can only have access to a single IAM role. That IAM role will be mapped to all of their resources. In the near term, administrators will be able grant saturn users access to multiple roles, which can be attached to resources.

## Creating Roles

Roles need to have the appropriate trust relationship in order to be used with Saturn Cloud. These are straightforward to create in the AWS console.

First, navigate to the [IAM role page in the AWS console](https://console.aws.amazon.com/iamv2/home?#/roles). Click on "Create Role"

<img width=300 src="/images/docs/create-role.png" alt-text="Create Role" class="doc-image-no-format"/>

Next, select "Web Identity", and choose the OIDC provider for your EKS cluster. Choose `sts.amazonaws.com` as the audience.

<img src="/images/docs/web-identity.png" alt-text="Create Role" class="doc-image"/>

At this point you can continue with role creation and permissions configuration as you would with any other IAM role.

If you want to construct the trust relationship manually, the policy should match this pattern:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::XXXX:oidc-provider/oidc.eks.us-west-2.amazonaws.com/id/XXXX"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.us-west-2.amazonaws.com/id/XXXX:aud": "sts.amazonaws.com"
        }
      }
    }
  ]
}
```

The value for "Federated" should be the ARN of your OIDC provider, and the `StringEquals` key should be the name of the identity provider.

## Attaching roles

After the role is created, you can attach the role to a particular user or group within Saturn Cloud. Go to the **Users & Groups** page, click the edit button for the the user or group you want to edit and choose an option for **IAM Role**.

<img src="/images/docs/iam-user.png" alt-text="User IAM role input" class="doc-image">