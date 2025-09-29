# API Tokens in Saturn Cloud

Saturn Cloud relies on API tokens to grant programmatic access to Saturn Cloud.

## Scoped tokens vs Un-scoped tokens

Saturn Cloud tokens can be scoped or un-scoped. Scoped tokens grant access to specific resources. Unscoped tokens can perform any action as the user or group that generated them.

{{% alert %}}
If you were using Saturn Cloud API tokens with Saturn Cloud versions 2024.05.01 and earlier, and you want the current Saturn Cloud tokens to behave the same way, simply choose unscoped tokens.
{{% /alert %}}
## User Tokens vs Resource Tokens

Saturn Cloud has 2 types of tokens. User tokens are generaly intended to be used when you are connecting to Saturn Cloud from an external source. Resource tokens are automatically embedded into Saturn Cloud resources under the environment variable "SATURN_TOKEN". To create user tokens

### User Tokens

1. Access the settings page by clicking on your username in the sidebar, then select "User Profile". Under the section "Access Keys" click on "New Token"

<img src="/images/api-tokens-user-guide.png" alt="User Profile page showing Access Keys section with New Token button" class="doc-image">

2. Make sure you copy the token value. Once the token is generated, you cannot retrieve this value.

### Resource Tokens

Resource tokens are automatically generated and embedded under the `SATURN_TOKEN` environment variable in your workspace. If you click on the "Manage" tab of your workspace, you can modify your token and change the scope, for example.

## Using tokens

Saturn Cloud expects tokens to be passed via HTTP headers by adding a header key of `Authorization` with a value of `token {TOKEN}`. The following Python code demonstrates how to use Tokens to authenticate with Saturn Cloud


```python
import requests
import os

url = "https://hellosaturn-deploy.community.saturnenterprise.io"
TOKEN = # your token
response = requests.get(
    url,
    headers={"Authorization": f"token {TOKEN}"}
)
```
