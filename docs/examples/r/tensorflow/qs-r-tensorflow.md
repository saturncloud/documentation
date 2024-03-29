# Train a TensorFlow model with a GPU in R



## Overview

R has the capability to train models with TensorFlow and Keras using packages
developed by RStudio. On the backend, these packages are running TensorFlow in
Python, and then the reticulate package converts the Python objects to R. As an
R user, that is largely abstracted away so you can primarily think in term of
the R language you're used to.

While often getting Python, R, TensorFlow, and the GPU drivers to all be the
correct versions and work together, Saturn Cloud provides a convenient
`saturn-rstudio-tensorflow` image that is preconfigured to train the models on a
GPU. If you'd rather train a model on a CPU, you can use the `saturn-rstudio`
image and install both the Python and R packages for Keras and TensorFlow.

## Example

In this example we'll be using pet names data from the city of Seattle and
training a Keras neural network to generate new names. Note that if instead
of using Keras you'd prefer to use pure TensorFlow you can directly use the 
tensorflow R package instead of the Keras one.

### Setup

The `saturn-rstudio-tensorflow` image has the required libraries
preinstalled--you just need to import them.

```{r libraries}
library(dplyr)
library(readr)
library(stringr)
library(purrr)
library(tidyr)
library(keras)
```

Define what characters can be used for the pet names, and how far back the
neural network should look when generating them.

```{r lookup_tables}
character_lookup <- data.frame(character = c(letters, ".", "-", " ", "+"))
character_lookup[["character_id"]] <- seq_len(nrow(character_lookup))

max_length <- 10
num_characters <- nrow(character_lookup) + 1
```

Finally, download the raw data and format it into a table

```{r cleaned_data}
data_url <-
  "https://saturn-public-data.s3.us-east-2.amazonaws.com/pet-names/seattle_pet_licenses.csv"
pet_data <-
  read_csv(data_url,
    col_types = cols_only(
      `Animal's Name` = col_character(),
      Species = col_character(),
      `Primary Breed` = col_character(),
      `Secondary Breed` = col_character()
    )
  ) %>%
  rename(
    name = `Animal's Name`,
    species = `Species`,
    primary_breed = `Primary Breed`,
    secondary_breed = `Secondary Breed`
  ) %>%
  mutate_all(toupper) %>%
  filter(!is.na(name), !is.na(species)) %>%
  filter(!str_detect(name, "[^ \\.-[a-zA-Z]]")) %>%
  mutate_all(stringi::stri_trans_tolower) %>%
  filter(name != "") %>%
  mutate(id = row_number())
```


### Create training data

Next, we take the downloaded data and modify it so it's ready for the model. 
First we add stop characters to signify the end of the name ("+"), expand
the names into sub-sequences so we can predict each character in the name.


```{r subsequence_data}
subsequence_data <-
  pet_data %>%
  mutate(
    accumulated_name =
      name %>%
        str_c("+") %>%
        str_split("") %>%
        map(~ purrr::accumulate(.x, c))
  ) %>%
  select(accumulated_name) %>%
  unnest(accumulated_name) %>%
  arrange(runif(n())) %>%
  pull(accumulated_name)
```

Then we make all the sequences the same length by truncating or padding them
so they can be a matrix. We also 1-hot encode the data.

```{r data_matrix}
text_matrix <-
  subsequence_data %>%
  map(~ character_lookup$character_id[match(.x, character_lookup$character)]) %>%
  pad_sequences(maxlen = max_length + 1) %>%
  to_categorical(num_classes = num_characters)
```

Once that's complete, we split the data into the 3D-matrix of model input (X) and
matrix of targets (y). We'll make the X matrix of all the letters in each row except
the last. The y matrix will be the last character (since we want to predict it).

```{r split_x_y}
x_name <- text_matrix[, 1:max_length, ]
y_name <- text_matrix[, max_length + 1, ]
```

### Create the model

Next we define the layers of the Keras model. This model has 2 LSTM layers to
find the patterns in the names, a dense layer to predict a value for each
possible next character, and a softmax activation to turn
those values into probabilities. Since this is a multiclass classification problem,
the loss is categorical cross-entropy.

```{r define_model}

input <- layer_input(shape = c(max_length, num_characters))

output <-
  input %>%
  layer_lstm(units = 32, return_sequences = TRUE) %>%
  layer_lstm(units = 32, return_sequences = FALSE) %>%
  layer_dropout(rate = 0.2) %>%
  layer_dense(num_characters) %>%
  layer_activation("softmax")

model <- keras_model(inputs = input, outputs = output) %>%
  compile(
    loss = "categorical_crossentropy",
    optimizer = "adam"
  )
```

### Train the model

Once the model is defined, we can train it.

```{r train_model}
fit_results <- model %>%
  fit(
    x_name,
    y_name,
    batch_size = 32768,
    epochs = 200
  )
```

### Generate names

The function below generates a pet name using the trained model.

```{r generate_names}
generate_name <- function(model, character_lookup, max_length, temperature = 1) {
  choose_next_char <- function(preds, character_lookup, temperature) {
    preds <- log(preds) / temperature
    exp_preds <- exp(preds)
    preds <- exp_preds / sum(exp(preds))
    next_index <- which.max(as.integer(rmultinom(1, 1, preds)))
    character_lookup$character[next_index - 1]
  }

  in_progress_name <- character(0)
  next_letter <- ""

  while (next_letter != "+" && length(in_progress_name) < 30) {
    previous_letters_data <-
      lapply(list(in_progress_name), function(.x) {
        character_lookup$character_id[match(.x, character_lookup$character)]
      })
    previous_letters_data <- pad_sequences(previous_letters_data,
      maxlen = max_length
    )
    previous_letters_data <- to_categorical(previous_letters_data,
      num_classes = num_characters
    )

    next_letter_probabilities <-
      predict(model, previous_letters_data)

    next_letter <- choose_next_char(
      next_letter_probabilities,
      character_lookup,
      temperature
    )

    if (next_letter != "+") {
      in_progress_name <- c(in_progress_name, next_letter)
    }
  }

  raw_name <- paste0(in_progress_name, collapse = "")

  capitalized_name <- gsub("\\b(\\w)", "\\U\\1", raw_name, perl = TRUE)

  capitalized_name
}
```

You can then generate a name by calling the function:

```{r generate_one_name}
generate_name(model, character_lookup, max_length)
```

Or generate many names at once:

```{r generate_many_names}
sapply(1:20, function(x) generate_name(model, character_lookup, max_length))
```

This will give you fun outputs like:

```
> sapply(1:20,function(x) generate_name(model, character_lookup, max_length))

 [1] "Poebwert" "Catera"   "Annie"    "Ikko"     "Spolly"   "Loly"    
 [7] "Blue"     "Charlie"  "Lucoi"    "Olivel"   "Clam"     "Coky"    
[13] "Feonne"   "Buster"   "Coco"     "Emma"     "Ree"      "Puns"    
[19] "Teko"     "Pocy"  
```

Notice that the names generated may be ones that are also in the original
training data. For true originality you may want to filter those out.

## Conclusion

Using R, Keras, and a GPU together is straightforward on Saturn Cloud. In
addition to model training, you could deploy the model as a Plumber API or
host it as an interactive Shiny app using Saturn Cloud deployments.

#### Acknowledgements

* The Rocker project for maintaining the R docker images this builds from.
* The RStudio developers for creating the keras, tensorflow, and reticulate packages.
* The City of Seattle for making the pet license data available for public use.

From the City of Seattle on the pet license data:

> The data made available here has been modified for use from its original
> source, which is the City of Seattle. Neither the City of Seattle nor the
> Office of the Chief Technology Officer (OCTO) makes any claims as to the
> completeness, timeliness, accuracy or content of any data contained in this
> application; makes any representation of any kind, including, but not limited
> to, warranty of the accuracy or fitness for a particular use; nor are any such
> warranties to be implied or inferred with respect to the information or data
> furnished herein. The data is subject to change as modifications and updates
> are complete. It is understood that the information contained in the web feed
> is being used at one's own risk.

#### Appendix: Run on a CPU

To instead use a cpu make the following changes to the Saturn Cloud resource:

* Switch to using the `saturn-rstudio` image.
* Add `keras` as an CRAN Extra Packages for the resource. This will install the R Keras 
  and TensorFlow packages.
* Add `pip install tensorflow` as a line in the startup script option of the resource. This will
  install the Python Keras and TensorFlow packages.
