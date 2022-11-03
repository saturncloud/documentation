# Train a Torch model with a GPU in R

As an equivalent to PyTorch for Python, R users can train GPU models using the
torch package from RStudio. Saturn Cloud provides the `saturn-rstudio-torch`
docker image that has the required libraries to use a GPU and torch. This image
is based on the `rocker/ml` R image from the Rocker team. 

## Example

In this example we'll be using pet names data from the city of Seattle and training a
torch neural network to generate new names.

### Setup

The `saturn-rstudio-torch` image has the required libraries
preinstalled--you just need to import them.

```{r libraries}
library(dplyr)
library(readr)
library(stringr)
library(purrr)
library(tidyr)
library(torch)
library(glue)
library(progress)
```

Next, to use the GPU we'll set the device for torch to use `cuda`. See the appendix at the
bottom for running on a CPU.

```{r}
device <- torch_device("cuda")
```

Define what characters can be used for the pet names, and how far back the
neural network should look when generating them.

```{r lookup_tables}
character_lookup <- data.frame(character = c(letters, ".", "-", " ", "+"))
character_lookup[["character_id"]] <- as.integer(seq_len(nrow(character_lookup)))
max_length <- 10
num_characters <- nrow(character_lookup) + 1
```

Finally, download the raw data and format it into a table

```{r cleaned_data}
data_url <-
  "https://saturn-public-data.s3.us-east-2.amazonaws.com/pet-names/seattle_pet_licenses.csv"
pet_raw_data <-
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
  pet_raw_data %>%
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

Then we make a matrix out of the subsequences by padding/concatenating them
to all be the same length, then binding the rows together. Once that matrix is
created, we can then one-hot encode it for the _X_-matrix of training data and
use the last column for the _y_-vector of training data. The training data is
also converted into torch tensors.

```{r create_tensors}
pad_sequence_single <- function(seq, maxlen) {
  diff <- length(seq) - maxlen
  if (diff < 0) {
    c(rep(0L, abs(diff)), seq)
  } else if (diff > 0) {
    seq[-seq_len(abs(diff))]
  } else {
    seq
  }
}

# converted characters to numbers then made into a matrix padded with zeros
text_matrix <-
  subsequence_data %>%
  map(~ character_lookup$character_id[match(.x, character_lookup$character)]) %>%
  map(pad_sequence_single, maxlen = max_length + 1) %>%
  do.call(rbind, .)

# the X-matrix of training data as a tensor
data_x <-
  (text_matrix + 1L) %>%
  torch_tensor(device = device) %>%
  {{ .[, 1:max_length] }} %>%
  nnf_one_hot(num_characters + 1) %>% # one hot encoding using a torch function
  {{ .$to(dtype = torch_float()) }}

# the y-vector of training data
data_y <-
  text_matrix[, max_length + 1] %>%
  torch_tensor(device = device)
```

### Create the model

Next we define the network for the torch mdoel. This model has 2 LSTM layers to
find the patterns in the names, a dense layer to predict a value for each
possible next character. Note that these are not returning probabilities because
the loss function converts them to probabilities when computing the loss.

```{r define_model}
network <- nn_module(
  "PetNameNetwork",
  initialize = function() {
    self$num_layers <- 2
    self$hidden_size <- 32

    self$lstm <- torch::nn_lstm(
      input_size = num_characters + 1,
      hidden_size = self$hidden_size,
      num_layers = self$num_layers,
      batch_first = TRUE,
      dropout = 0.1
    )
    self$fc <- nn_linear(self$hidden_size, num_characters)
  },
  forward = function(x) {
    result <- self$lstm(x)
    hidden <- result[[2]][[2]]
    self$fc(hidden[self$num_layers, ])
  }
)
```

Set the variables for training, create the model and define the optimizer and
loss function. 

```{r}
batch_size <- 2048
num_epochs <- 50

num_data_points <- data_y$size(1)
num_batches <- floor(num_data_points / batch_size)

model <- network()$to(device = device)

optimizer <- optim_adam(model$parameters)
criterion <- nn_cross_entropy_loss()
```

### Train the model

Once the model is defined, we can train it. For convenience here `glue` and
`progress` are used to monitor how the training is going.

**NOTE:** notice that we are not using a 
[`dataset`](https://torch.mlverse.org/docs/reference/dataset.html) or 
[`dataloader`](https://torch.mlverse.org/docs/reference/dataloader.html) for
this model. Instead we are manually sorting and pulling batches from the data.
This is because as of the 0.6.0 version of the torch package there are
performance penalties for using those functions. In the interest having the
model train as quickly as possible we do not use them. If you chose to use a
`dataset` the training will still go faster if you use the
[`.getbatch()`](https://torch.mlverse.org/docs/reference/dataloader.html)
command instead of `.getitem()`.

```{r train_model}
for (current_epoch in 1:num_epochs) {
  pb <- progress::progress_bar$new(
    total = num_batches,
    format = glue("[:bar] Epoch {current_epoch} :percent eta: :eta")
  )

  permute <- torch_randperm(num_data_points) + 1L
  data_x <- data_x[permute]
  data_y <- data_y[permute]

  for (batch_idx in 1:num_batches) {
    batch <- (batch_size * (batch_idx - 1) + 1):(batch_idx * batch_size)

    optimizer$zero_grad()
    output <- model(data_x[batch])
    loss <- criterion(output, data_y[batch])
    loss$backward()
    optimizer$step()

    pb$tick()
  }
  message(glue::glue("Epoch {current_epoch} complete, loss: {loss$item()}"))
}
```

### Generate names

The function below generates a pet name using the trained model.

```{r generate_names}
generate_name <- function(model, character_lookup, max_length, temperature = 1) {
  choose_next_char <- function(raw_preds, character_lookup, temperature) {
    preds <-
      raw_preds %>%
      nnf_softmax(dim = 2) %>%
      {{ .$to(device = "cpu") }} %>%
      as_array()
    preds <- log(preds) / temperature
    preds <- exp(preds) / sum(exp(preds))
    next_index <- which.max(as.integer(rmultinom(1, 1, preds)))
    character_lookup$character[next_index]
  }

  in_progress_name <- character(0)
  next_letter <- ""

  while (next_letter != "+" && length(in_progress_name) < 30) {
    text_matrix <-
      in_progress_name %>%
      {{ character_lookup$character_id[match(., character_lookup$character)] }} %>%
      pad_sequence_single(maxlen = max_length) %>%
      matrix(nrow = 1)

    data_x <- (text_matrix + 1L) %>%
      torch_tensor(device = device) %>%
      {{ .[, 1:max_length] }} %>%
      nnf_one_hot(num_characters + 1) %>%
      {{ .$to(dtype = torch_float()) }}


    next_letter_probabilities <- model(data_x)

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

Using R, torch, and a GPU together is straightforward on Saturn Cloud. In
addition to model training, you could deploy the model as a Plumber API or
host it as an interactive Shiny app using Saturn Cloud deployments.

#### Acknowledgements

* The Rocker project for maintaining the R docker images this builds from.
* Daniel Falbel, RStudio, and the other developers for creating the torch package.
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

To instead use a cpu make the following changes:

* In the `device <- torch_device("cuda")` chunk change `cuda` to `cpu` 
* In the Saturn Cloud resource settings make the following changes:
  * Switch to using the `saturn-rstudio` image
  * Add `torch` as an CRAN Extra Package for the resource
  * Add `Rscript -e "torch::install_torch()"` as a line in the startup
  script option of the resource
