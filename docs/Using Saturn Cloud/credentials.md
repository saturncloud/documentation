# Add Credentials

You may need to have secret credentials in your working environment in order to access tools or data. For specific instructions on setting this up for Snowflake or S3 data storage, please see the [Connect Saturn Cloud to your Data](<docs/Using Saturn Cloud/connect_data.md>) page.

For general instructions on adding a new credential, please see below!

***

Open the Saturn Cloud platform, and click the "Tools" section, and select "Credentials".

<img src="/images/docs/creds1.png" alt="Screenshot of side menu of Saturn Cloud product with Credentials selected" style="width:150px;" class="doc-image">

This is where you will store your credential information. *This is a secure storage location, and will not be available to the public or other users without your consent.*

At the top right corner of this page, you will find the "Create" button. Click here, and you'll be taken to the Credentials Creation form. This will give you some options, starting with the type of credential this is.

<img src="/images/docs/creds2.jpg" alt="Screenshot of Saturn Cloud Create Credentials form" class="doc-image">

There are 4 types of credentials:

* **Environment Variable**: This can cover many use cases. For example, AWS S3 credentials and Snowflake credentials will be environment variables. Specify the name of the credential, the name of the environment variable (how you will refer to it in your code), and the value.
* **File**: This indicates you wish to upload a whole file as a secret credential. Specify the name of the credential, the path to the file, and contents of the file.
* **SSH Public Key**: specify the name of the credential, and paste in the ssh public key for your private key, and it will be added to ~/.ssh/authorized_keys so that you can ssh into your Jupyter server.
* **SSH Private Key**: specify the name of the credential, the path for the private key, and paste in the ssh private key for use with private repository authentication, for example.

Complete the form one time for each credential you wish to add. When completed, you should see the credentials now in the Credentials page, like this.

<img src="/images/docs/creds3.png" alt="Screenshot of Credentials list in Saturn Cloud product" class="doc-image">

When this is all complete, you can start a project and your credentials will be set.