# Load Data from Kaggle|
| Kaggle Username  |  Environment Variable | `kaggle-username`  | `KAGGLE_USERNAME`  | `<your username>`
| Kaggle API Key  |  Environment Variable | `kaggle-api-key`  | `KAGGLE_KEY`  | `<your key>`

Copy the values (without quotes) from your “kaggle.json” file into the **Value** section of the credential creation form. The credential names are recommendations; feel free to change them as needed for your workflow. You must, however, use the provided **Variable Names** for Kaggle to connect correctly.

With this complete, your Kaggle credentials will be accessible by Saturn Cloud resources!

## Set Up Your Saturn Cloud Environment
You can use your Kaggle connection in any resource, but to test everything out, we can create the following small resource. In Saturn Cloud, go to the **Resources** page and create a new Jupyter server. Choose the following settings:

- **Name**: `kaggle-dataset-download`
- **Disk Space**: 10 Gi
- **Hardware**: CPU
- **Size**: Large - 2 cores - 16 GB RAM
- **Image**: saturncloud/saturn - 2021.08.16
- **Extra Packages - Pip Install**:
    - ```
      kaggle
      ```

![Saturn Cloud left menu with arrow pointing to Credentials tab](/images/docs/kaggle_resource_definition.jpeg "doc-image")

## Download a Dataset
Now that you have set up the credentials for Kaggle properly, downloading Kaggle data is really straightforward! 

In Kaggle, find the dataset you want to download. 

On the dataset page, click on the three dots to the right and select **Copy API Command**.

![Kaggle dataset page with arrow pointing to Copy API command](/images/docs/kaggle_dataset_arrow.jpeg "doc-image")

Now, in Saturn Cloud, create a new notebook (or open an existing one). In a cell, enter “**!**” and then paste the API command.

```  
! kaggle datasets download -d deepcontractor/swarm-behaviour-classification
```

That’s it! Your dataset will download to your current path, and you will be able to use it for calculations!

## Download a Competition Dataset
Downloading a competition dataset is similarly straightforward, but it is a slightly different process. 

In Kaggle, find the competition you want to download the dataset for.

Click on **Data** in the top menu and then copy the command displayed. 

![Kaggle competition dataset page with arrows pointing to the Data tab and the API command](/images/docs/kaggle_competition_dataset_arrow.jpeg "doc-image")

Now, in Saturn Cloud, create a new notebook (or open an existing one). In a cell, enter “**!**” and then paste the API command.

```  
! kaggle competitions download -c titanic
```
That’s it! Your dataset will download to your current path, and you will be able to use it for calculations!

## Conclusion
In this example, we showed how to download data directly from Kaggle to use in your Saturn Cloud notebooks. Check out our other [examples](/docs) for how to do [machine learning](/docs) using this and other data! 

Want to download data from other sources? Check out our examples for downloading data from [s3 buckets](<docs/Examples/LoadData/load_data_s3.md>) and [Snowflake](<docs/Examples/LoadData/qs-snowflake-dask.md>).