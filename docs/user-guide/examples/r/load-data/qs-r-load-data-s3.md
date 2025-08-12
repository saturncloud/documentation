# Load Data From S3 Buckets|
| AWS Access Key ID   |  Environment Variable  | `aws-access-key-id` | `AWS_ACCESS_KEY_ID`
| AWS Secret Access Key | Environment Variable  | `aws-secret-access-key`  | `AWS_SECRET_ACCESS_KEY`
| AWS Default Region  | Environment Variable  | `aws-default-region`  | `AWS_DEFAULT_REGION`

Copy the values from your AWS console into the *Value* section of the credential creation form. The credential names are recommendations; feel free to change them as needed for your workflow. You must, however, use the provided *Variable Names* for S3 to connect correctly.

With this complete, your S3 credentials will be accessible by Saturn Cloud resources! You will need to restart any RStudio Server for the credentials to populate to those resources.

<a id='connect-via-aws-s3'></a>

### Setting Up Your Resource
`aws.s3` is not installed by default in Saturn images, so you will need to install it onto your resource. This is already done in this example recipe, but if you are using a custom resource you will need to `install.packages("aws.s3")`. Check out our page on [installing packages](https://saturncloud.io/docs/user-guide/how-to/install-packages/) to see the various methods for achieving this!

### Connect to Data Via `aws.s3`
#### Set Up the Connection
Normally, `aws.s3` will automatically seek your AWS credentials from the environment. Since you have followed our instructions above for adding and saving credentials, this will work for you! The below command simply lists your s3 buckets.

```{r}
library(aws.s3)

bucketlist()
```

Now you can save files directly to your current working directory using the `save_object` command.

```{r }
save_object("s3://saturn-public-data/hello_world.txt")
```

If you prefer to read the data directly into a variable (in this instance, a data.table), you can save it as a temp file and read it from there.

```{r}
library(dplyr)
library(data.table)

data <-
  save_object("s3://saturn-public-data/pet-names/seattle_pet_licenses.csv",
    file = tempfile(fileext = ".csv")
  ) %>%
  fread()
```
