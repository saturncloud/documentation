# Load Data From Snowflake|
| Snowflake account   | Environment Variable  | `snowflake-account` | `SNOWFLAKE_ACCOUNT` 
| Snowflake username | Environment Variable  |`snowflake-user`  | `SNOWFLAKE_USER`
| Snowflake user password  | Environment Variable  |`snowflake-password`  | `SNOWFLAKE_PASSWORD`

Enter your values into the *Value* section of the credential creation form. The credential names are recommendations; feel free to change them as needed for your workflow.

If you are having trouble finding your Snowflake account id, it is the first part of the URL you use to sign into Snowflake. If you use the url `https://AA99999.us-east-2.aws.snowflakecomputing.com/console/login` to login, your account id is `AA99999`.

With this complete, your Snowflake credentials will be accessible by Saturn Cloud resources! You will need to restart any RStudio Server for the credentials to populate to those resources.

### Setting Up Your Resource
`odbc` is not installed by default in Saturn images, so you will need to install it onto your resource. This is already done in this example recipe, but if you are using a custom resource you will need to `install.packages("odbc")`. Check out our page on [installing packages](https://saturncloud.io/docs/user-guide/using-saturn-cloud/install-packages/) to see the various methods for achieving this!

Additionally, Saturn Cloud images come with Linux Snowflake ODBC drivers pre-installed. If you are using this code outside of Saturn Cloud, you will need to install the appropriate drivers.

### Connect to Data

From a RStudio resource where you want to connect to Snowflake, you can use the credentials as environment variables and provide any additional arguments, if necessary.


```{r}
library(DBI)

con <- dbConnect(odbc::odbc(),
  driver = "SnowflakeDSIIDriver",
  server = paste0(Sys.getenv("SNOWFLAKE_ACCOUNT"), ".us-east-2.aws.snowflakecomputing.com"),
  uid = Sys.getenv("SNOWFLAKE_USER"),
  pwd = Sys.getenv("SNOWFLAKE_PASSWORD")
)
```

If you changed the *variable name* of any of your credentials, simply change them here for them to populate properly.

In RStudio, the connection will now appear in the **Connections** pane, along with a list of available databases.

> **Note**: A running warehouse is required to actually access any data.

Now you can simply query the database as you would on a local machine.

```
