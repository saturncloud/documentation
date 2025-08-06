# Okta

To use Okta to authenticate Saturn Cloud Enterprise, use the following steps:

1. Login to your Okta account and navigate to the Okta dashboard. In the sidebar, click on "Applications"

<img width=200 src="/images/docs/okta-sidebar.webp" alt="Okta Side Bar" class="doc-image-no-format">

2. Choose *Create App Integration*
<img width=500 src="/images/docs/okta-create-app.webp" alt="Okta Create App" class="doc-image-no-format">

3. Select *OIDC* and *Web Application*

<img width=500 src="/images/docs/okta-create-app-2.webp" alt="Okta Create App Form" class="doc-image-no-format">

4. In the resulting form, set the sign-in redirect URI (which we will provide to you separately). For Grant type allowed, we require openid, email, profile, and groups.

<img width=600 src="/images/docs/okta-sign-in-uri.webp" alt="Okta Set Sign In URI" class="doc-image-no-format">

5. Under assignments, choose *Allow everyone in your organization to access*. or *Limit access to specific groups*. Saturn Cloud has additional controls for adding new users, so you do not have to be completely precise here. For simplicity We recommend *Allow everyone in your organization to access*.

<img width=600 src="/images/docs/okta-everyone-access.webp" alt="Okta Every Access" class="doc-image-no-format">

6. Click *Save*. Afterwards, click on *Sign On* in order to configure *Sign On* options.

<img width=600 src="/images/docs/okta-signon.webp" alt="Selct Okta Sign On" class="doc-image-no-format">

7. Click to edit the *Open ID Connect Token*. Modify the selector to *Matches Regex* and then use `.*` as the value. This ensures that Saturn Cloud gets all group membership information, which Saturn Cloud admins can use to control entitlements within Saturn Cloud.

<img width=600 src="/images/docs/okta-groups.webp" alt="Okta Groups" class="doc-image-no-format">

8. Please Store the client ID and Secret for this application. We will invite you to your Auth0 tenant, where you can input this information securely.
{{% enterprise_docs_view %}}
