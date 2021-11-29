# Load Data From Snowflake|
| Snowflake Account   | Environment Variable  | `snowflake-account` | `SNOWFLAKE_ACCOUNT` 
| Snowflake Username | Environment Variable  |`snowflake-user`  | `SNOWFLAKE_USER`
| Snowflake User Password  | Environment Variable  |`snowflake-password`  | `SNOWFLAKE_PASSWORD`

Enter your values into the *Value* section of the credential creation form. The credential names are recommendations; feel free to change them as needed for your workflow.

If you are having trouble finding your Snowflake account id, it is the first part of the URL you use to sign into Snowflake. If you use the url `https://AA99999.us-east-2.aws.snowflakecomputing.com/console/login` to login, your account id is `AA99999`.

With this complete, your Snowflake credentials will be accessible by Saturn Cloud resources! You will need to restart any Jupyter Server or Dask Clusters for the credentials to populate to those resources.

### Connect to Data

From a notebook where you want to connect to Snowflake, you can use the credentials as environment variables and provide any additional arguments, if necessary.


```python
import os

import snowflake.connector

conn_info = {
    "account": os.environ["SNOWFLAKE_ACCOUNT"],
    "user": os.environ["SNOWFLAKE_USER"],
    "password": os.environ["SNOWFLAKE_PASSWORD"],
    "warehouse": "MY_WAREHOUSE",
    "database": "MY_DATABASE",
    "schema": "MY_SCHEMA",
}

conn = snowflake.connector.connect(**conn_info)
```

If you changed the *variable name* of any of your credentials, simply change them here for them to populate properly.

Now you can simply query the database as you would on a local machine.
