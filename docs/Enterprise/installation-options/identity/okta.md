# Okta

To use Okta to authenticate Saturn Cloud Enterprise, use the following steps:

1. Login to your Okta account and navigate to the Okta dashboard.
2. Select *Applications*, then *Add Application*.
3. Select "Web Application"
4. Set *Login Redirect URI* to your Auth0 callback URI (which we will give you). It will be something like this: https://your-company.us.auth0.com/login/callback
5. For *Grant type allowed*, we require openid, email, and profile
6. Okta will provide you with a client ID and secret. Give those to us, and we will complete the Auth0 configuration.
