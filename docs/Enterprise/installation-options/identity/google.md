# Google Authentication

To use Google Authentication to authenticate Saturn Cloud Enterprise, use the following steps:

1. Navigate to [Google Credentials API](https://console.cloud.google.com/apis/credentials) and click "Create Credentials"

<img src="/images/docs/google-create-creds.png" alt="create creds in google cloud" class="doc-image"/>

2. Select *Create OAuth Client ID*

3. under *Application Type* select *Web Application*

4. *Authorized JavaScript origins* can be ignored.

5. Under *Authorized redirect URIs*, enter your Auth0 callback URI (which we will give you). It will be something like this: https://your-company.us.auth0.com/login/callback

6. Google will provide you with a client ID and secret. Give those to us, and we will complete the Auth0 configuration.
