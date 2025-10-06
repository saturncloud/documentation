# Azure AD

To use Azure AD to log in to Saturn Cloud Enterprise, follow these steps:

## Step 1: Create an Application

[Create an application with the Microsoft identity platform](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app)

<img src="/images/docs/portal-app-reg-01.webp" class="doc-image" alt="app-reg"/>

## Step 2: Add Redirect URI

Under *Redirect URI*, enter your Auth0 callback URI. We will give you this URI. It will look like this:

```bash
https://your-company.us.auth0.com/login/callback
```

For example:

```bash
https://acme-corp.us.auth0.com/login/callback
```

## Step 3: Record the Client ID

Copy and save the Client/Application ID shown on the screen. It will look like this:

```bash
12345678-1234-1234-1234-123456789abc
```

## Step 4: Create a Secret

[Generate the secret](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app#add-credentials)

Copy and save the secret value. It will look like this:

```bash
a1B2c3D4e5F6g7H8i9J0k1L2m3N4o5P6q7R8s9T0
```

## Step 5: Send Information to Saturn Cloud

Send us your Client ID and secret. We will finish setting up Auth0 for you.

{{% enterprise_docs_view %}}
