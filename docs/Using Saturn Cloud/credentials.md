# Add Credentials to Saturn Cloud

You may need to have secret credentials in your working environment in order to access tools or data.  Saturn Cloud provides a general method for setting up secret environment variables and files. For specific instructions on setting this up for Snowflake or S3 data storage, please see the [Connect Saturn Cloud to your Data](<docs/Using Saturn Cloud/connect_data.md>) page.

To add a credential open the **Credentials** page in Saturn Cloud.

<img src="/images/docs/creds1.png" alt="Screenshot of side menu of Saturn Cloud product with Credentials selected" style="width:150px;" class="doc-image">

This is where you will store your credential information. *This is a secure storage location, and will not be available to the public or other users.*

At the top right corner of this page, you will find the "Create" button. Click here, and you'll be taken to the Credentials Creation form. This will give you some options, starting with the type of credential this is.

<img src="/images/docs/creds2.jpg" alt="Screenshot of Saturn Cloud Create Credentials form" class="doc-image">

There are 3 types of credentials:

* **Environment Variable**: This can cover many use cases. For example, AWS S3 credentials and Snowflake credentials will be environment variables. Specify the name of the credential, the name of the environment variable (how you will refer to it in your code), and the value.
* **File**: This indicates you wish to upload a whole file as a secret credential. Specify the name of the credential, the path to the file, and contents of the file.
* **SSH Public Key**: This special type of credential is used for [connecting via SSH](<docs/Using Saturn Cloud/ide_ssh.md>) to a resource.

Complete the form one time for each credential you wish to add. When completed, you should see the credentials now in the Credentials page, like this.

<img src="/images/docs/creds3.png" alt="Screenshot of Credentials list in Saturn Cloud product" class="doc-image">

When this is all complete, you can start a project and your credentials will be set.
