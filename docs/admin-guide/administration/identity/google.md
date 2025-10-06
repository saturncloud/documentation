# Google Authentication

To use Google Authentication to authenticate Saturn Cloud Enterprise, use the following steps:

## Step 1: Open Google Cloud Credentials

Navigate to [Google Cloud → APIs & Services → Credentials](https://console.cloud.google.com/apis/credentials) and click "Create Credentials".

<img src="/images/docs/google-create-creds.webp" alt="create creds in google cloud" class="doc-image"/>

<img src="/images/choose-credentials-google-cloud-console.png" alt="Choose Create Credentials in Google Cloud Console" class="doc-image"/>

## Step 2: Configure OAuth Consent Screen (Optional)

If prompted, configure the OAuth consent screen:

- Select **Internal** (recommended) or **External** according to your organization policy.
- Under Scopes, include at minimum: `openid`, `email`, `profile`.
- Save and continue with the defaults unless your org requires additional fields.

## Step 3: Create OAuth Client ID

- Choose **Create OAuth client ID**.
- Set **Application type** to **Web application**.
- You can ignore **Authorized JavaScript origins** for this integration.

<img src="/images/choose-oauth-client-id-google-cloud-console.png" alt="Choose OAuth client ID" class="doc-image"/>

<img src="/images/choose-web-application-google-cloud-console.png" alt="Choose Web application type" class="doc-image"/>

## Step 4: Add Redirect URI

Under **Authorized redirect URIs**, enter your Auth0 callback URI (we will provide this). It will look like:

```bash
https://your-company.us.auth0.com/login/callback
```

For example:

```bash
https://acme-corp.us.auth0.com/login/callback
```

Save the client.

<img src="/images/authorized-redirect-uris-google-cloud-console.png" alt="Authorized redirect URIs configuration" class="doc-image"/>

## Step 5: Record Client ID and Secret

After saving, Google shows a Client ID and Client Secret. Copy both and keep them secure. Share them with us so we can complete the Auth0 configuration for your Saturn Cloud Enterprise tenant.

<img src="/images/oauth-client-created-google-cloud-console.png" alt="OAuth client created with client ID shown" class="doc-image"/>

{{% enterprise_docs_view %}}
