# Azure AD

To use Azure AD to authenticate Saturn Cloud Enterprise, use the following steps:

1. [Create an application with the Microsoft identity platform](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app)
2. Under *Redirect URI* , enter your Auth0 callback URI (which we will give you). It will be something like this: https://your-company.us.auth0.com/login/callback
3. Record the Client ID
4. [Generate the secret](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app#add-credentials).
5. Provide you with a client ID and secret. Give those to us, and we will complete the Auth0 configuration.
{{% enterprise_docs_view %}}
