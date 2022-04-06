# Using the furrr Package


## Overview
[Furrr](https://furrr.futureverse.org/) is an extension of the [purrr](https://purrr.tidyverse.org/) mapping functions using the future package's parallel backend. Furrr allows purrr functions like `map()` and `pmap()` to be replaced with `future_map()` and `future_pmap()`, respectively, to run the functions in parallel. 

To illustrate the benefits of this package, we will run a [XGBoost](https://xgboost.readthedocs.io/en/stable/index.html) regression model based on on a CDC dataset of U.S. birth records in 2005. We will use this data to answer the following question:

> Based on pregnancy characteristics, can we determine the birth weight of the baby?

By using the furrr package, we can iterate over hyperparameters in a parallel fashion, thereby reducing the computation time.

> **Note**: The overhead associated with moving data and setting up the parallel computing means that furrr is only suitable for manipulations involving long compute times. Often, simple operations can take significantly longer when using furrr instead of purrr. As always, test single-threaded first, then try multi-threaded if single-threaded doesn't prove sufficient. 

## Modeling Process
### Imports
This exercise uses a variety of packages to complete the machine learning model:

* [furrr](https://furrr.futureverse.org/): For running the calculations in parallel
* [xgboost](https://xgboost.readthedocs.io/en/stable/R-package/xgboostPresentation.html): For training the model
```{r imports}
library(Metrics)
library(furrr)
library(future)
library(readr)
library(rsample)
library(tictoc)
library(tidyverse)
library(xgboost)
```

### Set the `furrr` future environment
First, we define the execution environment for `furrr` to run in. In this case, we are using a multisession environment with eight processes for the eight cores in the system. You could also specify the number of processes you want to use (e.g., `plan(multisession(workers = 8))`).

``` {r set environment}
plan(multisession)
```


### Download and select the dataset
We need to define functions to download the dataset from s3 and select the appropriate columns.

```{r download filter and split data}
download_data <- function() {
    if (!file.exists("births_data.rds")) {
        download.file(
            "https://saturn-public-data.s3.us-east-2.amazonaws.com/birth-data/births_2005.rds",
            "births_data.rds"
        )
    }
    births_raw_data <- read_rds("births_data.rds")
}
filter_data <- function(births_raw_data) {
    births_data <- births_raw_data %>%
        select(weight_pounds, is_male, plurality, mother_age, gestation_weeks)
}
```

### Preprocess the data, split, and create DMatrices
Once we have the data, we can preprocess the data. Here we will simply remove NA values, but other preprocessing steps could be added in the future. We then take this preprocessed data and splits it into training and test sets. Finally, we have a function that converts this data into xgboost DMatrices.


```{r preprocess, split, and create matrices}
preprocess_data <- function(df) {
    df_preprocessed <- df %>%
        drop_na()
}

create_split <- function(data) {
    data_split <- initial_split(data, prop = 0.8)
}

create_matrices <- function(data) {
    train_test_split <- create_split(data)

    train_df <- training(train_test_split)
    test_df <- testing(train_test_split)

    train_data <- subset(train_df, select = -c(weight_pounds))
    test_data <- subset(test_df, select = -c(weight_pounds))

    dtrain <- xgb.DMatrix(
        data = as.matrix(train_data),
        label = train_df$weight_pounds
    )
    dtest <- xgb.DMatrix(
        data = as.matrix(test_data),
        label = test_df$weight_pounds
    )

    return(list("train" = dtrain, "test" = dtest))
}
```

### Train and Test the Model
Our last function definitions define and train the model and determine the quality of the results. The model is fit, the mean absolute error calculated, and the results added to a table with the corresponding hyperparameters.

> **Note**: We disable XGBoost's multi-threading in this example to show the difference between furrr and purrr for single-threaded workloads.

```{r train model}
train_model <- function(params, dtrain) {
    model <- xgb.train(
        params = params,
        data = dtrain,
        nrounds = 100,
        nthread = 1,
        objective = "reg:squarederror",
    )
}

test_results <- function(model, dtest) {
    results <- predict(model, dtest)
}

create_model_table <- function(data, ...) {
    dmatrices <- create_matrices(data)

    dtrain <- dmatrices$train
    dtest <- dmatrices$test

    params <- list(...)

    model <- train_model(params, dtrain)
    results <- test_results(model, dtest)
    return(
        tibble(
            mean_absolute_error = mae(getinfo(dtest, "label"), results),
            params = params
        )
    )
}
```

### Let's Run the Model!
It's finally time to run the functions we just created. We also define the hyperparameters we want to test. We use the expand_grid function (with `list()` around constants) to create the appropriate. Here we are only testing max_depth, but other hyperparameters could be added to the grid.

Here are all the preprocessing steps.

```{r preprocess}
births_raw_data <- download_data()
births_data <- filter_data(births_raw_data)
births_data_preprocessed <- preprocess_data(births_data)

params <- expand_grid(
    data = list(births_data_preprocessed),
    max_depth = seq(1, 8)
)
```

#### Try Running the Model with purrr
Let's first try it with purrr to iterate through the various models.

Actually, let's not -- it takes over 17 minutes to run this code.  
```{r purrr evaluation}

# ###################################################
# ## Don't actually run this - it takes forever... ##
# ###################################################

# message("With purrr:")
# tic()
# results_purrr <- pmap_dfr(params, create_model_table)
# toc()

# best_result_purrr <- results_purrr %>%
#     top_n(-1, mean_absolute_error) %>%
#     head(1)

# message(best_result_purrr)
```

#### Run the Model with furrr

Instead, let's try it with a parallel back-end. In this case, we ask for eight workers for eight processes.

The only change to the purrr code is to add the futures `plan` and exchanging `pmap_dfr` with `future_pmap_dfr`. 

```{r furrr evaluation}
#####################
## Run this instead ##
#####################

message("With furrr:")
tic()
results_furrr <- future_pmap_dfr(params, create_model_table, .options = furrr_options(seed = NULL))
toc()

best_result_furrr <- results_furrr %>%
    top_n(-1, mean_absolute_error) %>%
    head(1)

message(best_result_furrr)
```

Nice! That only took five minutes to complete the training of eight different models. That is three times faster than the purrr code!

Why isn't it eight times faster?

Despite this being a perfectly parallelizable algorithm, moving to a parallel backend involves tradeoffs. Setup and data communication take time. Because we are using four cores on an eight-core machine, each model trains slightly more slowly than a single model with more overhead. Additionally, because some of the models complete more quickly than other models, cores may be idle if there are no more calculations to complete. Running more models would smooth out some of this variability.

## Conclusion
Furrr is a great solution for speeding up parallelizable code. If the computations are extensive enough to overcome any slowdown due to setup and data communication, parallelizing using furrr over purrr can lead to significant time savings. 

Thanks to [Deep Learning With Keras To Predict Customer Churn](https://blogs.rstudio.com/ai/posts/2018-01-11-keras-customer-churn/) for the inspiration for this article.