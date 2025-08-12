# Load Data from Kaggle|
| Kaggle Username  |  Environment Variable | `kaggle-username`  | `KAGGLE_USERNAME`
| Kaggle API Key  |  Environment Variable | `kaggle-api-key`  | `KAGGLE_KEY`

Copy the values from your "kaggle.json" file into the *Value* section of the credential creation form. The credential names are recommendations; feel free to change them as needed for your workflow. You must, however, use the provided *Variable Names* for Kaggle to connect correctly.

With this complete, your Kaggle credentials will be accessible by Saturn Cloud resources! You will need to restart any Jupyter Server or Dask Clusters for the credentials to populate to those resources.

### Setting Up Your Resource
Kaggle is not installed by default in Saturn images, so you will need to install it onto your resource. This is already done in this example recipe, but if you are using a custom resource you will need to `pip install kaggle`. Check out our page on [installing packages](https://saturncloud.io/docs/user-guide/how-to/install-packages/) to see the various methods for achieving this!

### Download a Dataset
Now that you have set up the credentials for Kaggle and installed kaggle, downloading Kaggle data is really straightforward! 

In Kaggle, find the dataset you want to download. 

On the dataset page, click on the three dots to the right and select **Copy API Command**.

![Kaggle dataset page with arrow pointing to Copy API command](https://saturn-public-assets.s3.us-east-2.amazonaws.com/example-resources/kaggle-dataset-arrow.jpeg "doc-image")

Now, in Saturn Cloud, open the terminal, then paste the API command. For example:


```{bash}
kaggle datasets download -d deepcontractor/swarm-behaviour-classification
```

That's it! Your dataset will download to your current path, and you will be able to use it for calculations!

### Download a Competition Dataset
Downloading a competition dataset is similarly straightforward, but it is a slightly different process. 

In Kaggle, find the competition you want to download the dataset for.

Click on **Data** in the top menu and then copy the command displayed. 

![Kaggle competition dataset page with arrows pointing to the Data tab and the API command](https://saturn-public-assets.s3.us-east-2.amazonaws.com/example-resources/kaggle-competition-dataset-arrow.jpeg "doc-image")

Now, in Saturn Cloud, open the terminal, then paste the API command. For example:

```{bash}
kaggle competitions download -c titanic
```

That's it! Your dataset will download to your current path, and you will be able to use it for calculations!
