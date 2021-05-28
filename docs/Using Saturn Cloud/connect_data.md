# Connect Saturn Cloud to Your Data|
| Snowflake account   |  `snowflake-account` | `SNOWFLAKE_ACCOUNT`  | Environment Variable  |
| Snowflake username | `snowflake-user`  | `SNOWFLAKE_USER`  | Environment Variable  |
| Snowflake user password  | `snowflake-password`  | `SNOWFLAKE_PASSWORD`  | Environment Variable  |

Complete the form one time for each item. When completed, you should see the credentials now in the Credentials page, like this.

<img src="/images/docs/creds3.png" alt="Screenshot of Credentials list in Saturn Cloud product" class="doc-image">


After this is done, you will need to restart any Jupyter Server or Dask Clusters (if you add credentials while they are running). Then from a notebook where you want to connect to Snowflake, you can read in the credentials and then provide additional arguments if necessary:

```python
import os
import snowflake.connector

conn = snowflake.connector.connect(
    account=os.environ['SNOWFLAKE_ACCOUNT'],
    user=os.environ['SNOWFLAKE_USER'],
    password=os.environ['SNOWFLAKE_PASSWORD'],
    warehouse='MY_WAREHOUSE',
    database='MY_DATABASE',
    schema='MY_SCHEMA',
)
```

At this point, you can access your Snowflake database the same way you would on a laptop or local machine. For more information about working with Snowflake data storage, please see our Tutorial about loading data.

