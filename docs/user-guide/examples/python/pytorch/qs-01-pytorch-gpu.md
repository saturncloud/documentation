# Train a PyTorch model with a GPU on Saturn Cloud



## Overview
This example shows how you can use the power of a GPU to quickly train a neural network in Saturn Cloud. This code runs on a single GPU of a Jupyter server resource.

This is an example of a natural language processing neural network which is trained on Seattle pet license data to then generate new pet names. The model uses LSTM layers which are especially good at discovering patterns in sequences like text. The model takes a partially complete name and determines the probability of each possible next character in the name. Characters are randomly sampled from this distribution and added to the partial name until a stop character is generated and full name has been created. For more detail about the network design and use case, see our [Saturn Cloud blog post](https://saturncloud.io/blog/dask-with-gpus/) which uses the same network architecture.

## Model training

### Imports

This code mainly relies on PyTorch for most of the work, however there are a number of other packages needed for manipulating data and other tasks.


```python
import datetime
import json
import torch
import torch.nn as nn
import torch.optim as optim
import numpy as np
import urllib.request
from torch.utils.data import Dataset, DataLoader
```

### Preparing data

This code is used to get the data in the proper format in an easy to use class.

First, download the data and create the character dictionary


```python
with urllib.request.urlopen(
    "https://saturn-public-data.s3.us-east-2.amazonaws.com/examples/pytorch/seattle_pet_licenses_cleaned.json"
) as f:
    pet_names = json.loads(f.read().decode("utf-8"))

# Our list of characters, where * represents blank and + represents stop
characters = list("*+abcdefghijklmnopqrstuvwxyz-. ")
str_len = 8
```

Next, create a function that will take the pet names and turn them into the formatted tensors. The [Saturn Cloud blog post](https://saturncloud.io/blog/dask-with-gpus/) goes into more detail on the logic behind how to format the data.


```python
def format_training_data(pet_names, device=None):
    def get_substrings(in_str):
        # add the stop character to the end of the name, then generate all the partial names
        in_str = in_str + "+"
        res = [in_str[0:j] for j in range(1, len(in_str) + 1)]
        return res

    pet_names_expanded = [get_substrings(name) for name in pet_names]
    pet_names_expanded = [item for sublist in pet_names_expanded for item in sublist]
    pet_names_characters = [list(name) for name in pet_names_expanded]
    pet_names_padded = [name[-(str_len + 1) :] for name in pet_names_characters]
    pet_names_padded = [
        list((str_len + 1 - len(characters)) * "*") + characters for characters in pet_names_padded
    ]
    pet_names_numeric = [[characters.index(char) for char in name] for name in pet_names_padded]

    # the final x and y data to use for training the model. Note that the x data needs to be one-hot encoded
    if device is None:
        y = torch.tensor([name[1:] for name in pet_names_numeric])
        x = torch.tensor([name[:-1] for name in pet_names_numeric])
    else:
        y = torch.tensor([name[1:] for name in pet_names_numeric], device=device)
        x = torch.tensor([name[:-1] for name in pet_names_numeric], device=device)
    x = torch.nn.functional.one_hot(x, num_classes=len(characters)).float()
    return x, y
```

Finally, create a PyTorch data class to manage the dataset:


```python
class OurDataset(Dataset):
    def __init__(self, pet_names, device=None):
        self.x, self.y = format_training_data(pet_names, device)
        self.permute()

    def __getitem__(self, idx):
        idx = self.permutation[idx]
        return self.x[idx], self.y[idx]

    def __len__(self):
        return len(self.x)

    def permute(self):
        self.permutation = torch.randperm(len(self.x))
```

### Define the model architecture

This class defines the LSTM structure that the neural network will use;


```python
class Model(nn.Module):
    def __init__(self):
        super(Model, self).__init__()
        self.lstm_size = 128
        self.lstm = nn.LSTM(
            input_size=len(characters),
            hidden_size=self.lstm_size,
            num_layers=4,
            batch_first=True,
            dropout=0.1,
        )
        self.fc = nn.Linear(self.lstm_size, len(characters))

    def forward(self, x):
        output, state = self.lstm(x)
        logits = self.fc(output)
        return logits
```

### Train the model
We define a `train()` function that will do the work to train the neural network. This function should be called once and will return the trained model. It will use the `torch.device(0)` command to access the GPU.


```python
def train():
    num_epochs = 8
    batch_size = 4096
    lr = 0.001
    device = torch.device(0)

    dataset = OurDataset(pet_names, device=device)
    loader = DataLoader(dataset, batch_size=batch_size, shuffle=True, num_workers=0)

    model = Model()
    model = model.to(device)

    criterion = nn.CrossEntropyLoss()
    optimizer = optim.Adam(model.parameters(), lr=lr)

    for epoch in range(num_epochs):
        dataset.permute()
        for i, (batch_x, batch_y) in enumerate(loader):
            optimizer.zero_grad()
            batch_y_pred = model(batch_x)

            loss = criterion(batch_y_pred.transpose(1, 2), batch_y)
            loss.backward()
            optimizer.step()
        print(
            f"{datetime.datetime.now().isoformat()} - epoch {epoch} complete - loss {loss.item()}"
        )
    return model
```

The next block of code actually runs the training function and creates the trained model.


```python
model = train()
```

After each epoch you should see a line of output like:

`2021-02-23T22:00:36.394824 - epoch 0 complete - loss 1.745424509048462`

### Generating Names

To generate names, we have a function that takes the model and runs it over and over on a string, generating each new character until a stop character is met.


```python
def generate_name(model, characters, str_len):
    device = torch.device(0)
    in_progress_name = []
    next_letter = ""
    while not next_letter == "+" and len(in_progress_name) < 30:
        # prep the data to run in the model again
        in_progress_name_padded = in_progress_name[-str_len:]
        in_progress_name_padded = (
            list((str_len - len(in_progress_name_padded)) * "*") + in_progress_name_padded
        )
        in_progress_name_numeric = [characters.index(char) for char in in_progress_name_padded]
        in_progress_name_tensor = torch.tensor(in_progress_name_numeric, device=device)
        in_progress_name_tensor = torch.nn.functional.one_hot(
            in_progress_name_tensor, num_classes=len(characters)
        ).float()
        in_progress_name_tensor = torch.unsqueeze(in_progress_name_tensor, 0)

        # get the probabilities of each possible next character by running the model
        with torch.no_grad():
            next_letter_probabilities = model(in_progress_name_tensor)

        next_letter_probabilities = next_letter_probabilities[0, -1, :]
        next_letter_probabilities = (
            torch.nn.functional.softmax(next_letter_probabilities, dim=0).detach().cpu().numpy()
        )
        next_letter_probabilities = next_letter_probabilities[1:]
        next_letter_probabilities = [
            p / sum(next_letter_probabilities) for p in next_letter_probabilities
        ]

        # determine what the actual letter is
        next_letter = characters[
            np.random.choice(len(characters) - 1, p=next_letter_probabilities) + 1
        ]
        if next_letter != "+":
            # if the next character isn't stop add the latest generated character to the name and continue
            in_progress_name.append(next_letter)
    # turn the list of characters into a single string
    pet_name = "".join(in_progress_name).title()
    return pet_name
```

Finally, let's generate 50 names! Also let's remove any names that would have shown up in the training data since those are less fun.


```python
# Generate 50 names then filter out existing ones
generated_names = [generate_name(model, characters, str_len) for i in range(0, 50)]
generated_names = [name for name in generated_names if name not in pet_names]
print(generated_names)
```

After running the code above you should see a list of names like:

```python
['Moicu', 'Caspa', 'Penke', 'Lare', 'Otlnys', 'Zexto', 'Toba', 'Siralto', 'Luny', 'Lit',
'Bonhe', 'Mashs', 'Riys Wargen', 'Roli', 'Sape', 'Anhyyhe', 'Lorla', 'Boupir', 'Zicka',
'Muktse', 'Musko', 'Mosdin', 'Yapfe', 'Snevi', 'Zedy', 'Cedi', 'Wivagok Rayten', 'Luzia',
'Teclyn', 'Pibty', 'Cheynet', 'Lazyh', 'Ragopes', 'Bitt', 'Bemmen', 'Duuxy', 'Graggie',
'Rari', 'Kisi', 'Lvanxoeber', 'Bonu', 'Masnen', 'Isphofke', 'Myai', 'Shur', 'Lani', 'Ructli',
'Folsy', 'Icthobewlels', 'Kuet Roter']
```

## Conclusion

We have now trained a neural network using PyTorch on a GPU, and used it for inference! If we wanted to experiment with trying many different hyperparameters for the model we could [concurrently train models](<docs/user-guide/examples/python/pytorch/qs-02-pytorch-gpu-dask-multiple-models.md>) with different hyperparameters using distributed computing. We could also train a [single neural network over many GPUs at once](<docs/user-guide/examples/python/pytorch/qs-03-pytorch-gpu-dask-single-model.md>) with distributed computing via Dask.
