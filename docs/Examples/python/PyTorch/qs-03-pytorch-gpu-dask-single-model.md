# Training a PyTorch Model across a Dask Cluster



## Overview

Training a PyTorch model can potentially be sped up dramatically by having the training computations done on multiple GPUs across multiple workers. This relies on PyTorches [DistributedDataParallel (DDP)](https://pytorch.org/tutorials/intermediate/ddp_tutorial.html) module to take computing the values for each batch and spread them across multiple machines/processors. So each worker computes a part of the batch, and then they are all combined to determine the loss then optimize the nodes. If you kept a network training setup the exact same except tripled the number of GPUs with DDP, you would in practice be using a batch size that is 3x bigger than our original one. Be aware, not all networks benefit from having larger batch sizes, and using PyTorch across multiple workers adds the time it takes to pass the new values between each worker.

This example builds on the [introduction to PyTorch with GPU on Saturn Cloud](<docs/Examples/python/PyTorch/qs-01-pytorch-gpu.md>) example that trains a neural network to generate pet names. The model uses LSTM layers which are especially good at discovering patterns in sequences like text. The model takes a partially complete name and determines the probability of each possible next character in the name. Characters are randomly sampled from this distribution and added to the partial name until a stop character is generated and full name has been created. For more detail about the network design and use case, see our [Saturn Cloud blog post](https://saturncloud.io/blog/dask-with-gpus/) which uses the same network architecture.

_Alternatively, rather than having a Dask cluster be used to train a single PyTorch model very quickly you could have the Dask cluster train many models in parallel. We have a [separate example](<docs/Examples/python/PyTorch/qs-02-pytorch-gpu-dask-multiple-models.md>) for that situation._

## Model Training

### Imports

This code uses PyTorch and Dask together, and thus both libraries have to be imported. In addition, the `dask_saturn` package provides methods to work with a Saturn Cloud dask cluster, and `dask_pytorch_ddp` provides helpers when training a PyTorch model on Dask.


```python
import uuid
import datetime
import pickle
import json
import torch
import torch.nn as nn
import torch.optim as optim
import numpy as np
import urllib.request
from torch.utils.data import Dataset, DataLoader

from torch.nn.parallel import DistributedDataParallel as DDP
import torch.distributed as dist
from torch.utils.data.distributed import DistributedSampler
from dask_pytorch_ddp import dispatch, results
from dask_saturn import SaturnCluster
from dask.distributed import Client
from distributed.worker import logger
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

### Train the model with Dask and Saturn Cloud

Next we train the model in parallel over multiple workers using Dask and Saturn. We define the `train()` function that will be run on each of the workers. This has much of the same training code you would see in any PyTorch training loop, with a few key differences. The data is distributed with the DistributedSampler--now each worker will only have a fraction of the data so that together all of the workers combined see each data point exactly once in an epoch. The model is also wrapped in a `DDP()` function call so that they can communicate with each other. The `logger` is used to show intermediate results in the Dask logs for each worker, and the results handler `rh` is used to write intermediate values back to the Jupyter server.


```python
def train():
    num_epochs = 25
    batch_size = 16384

    worker_rank = int(dist.get_rank())
    device = torch.device(0)

    logger.info(f"Worker {worker_rank} - beginning")

    dataset = OurDataset(pet_names, device=device)
    # the distributed sampler makes it so the samples are distributed across the different workers
    sampler = DistributedSampler(dataset)
    loader = DataLoader(dataset, batch_size=batch_size, sampler=sampler)
    worker_rank = int(dist.get_rank())

    # the model has to both be passed to the GPU device, then has to be wrapped in DDP so it can communicate with the other workers
    model = Model()
    model = model.to(device)
    model = DDP(model)

    criterion = nn.CrossEntropyLoss()
    optimizer = optim.Adam(model.parameters(), lr=0.001)

    for epoch in range(num_epochs):
        # the logger here logs to the Dask log of each worker, for easy debugging
        logger.info(
            f"Worker {worker_rank} - {datetime.datetime.now().isoformat()} - Beginning epoch {epoch}"
        )

        # this ensures the data is reshuffled each epoch
        sampler.set_epoch(epoch)
        dataset.permute()

        # nothing in the code for each batch is now any different than base PyTorch
        for i, (batch_x, batch_y) in enumerate(loader):
            optimizer.zero_grad()
            batch_y_pred = model(batch_x)

            loss = criterion(batch_y_pred.transpose(1, 2), batch_y)
            loss.backward()
            optimizer.step()

            logger.info(
                f"Worker {worker_rank} - {datetime.datetime.now().isoformat()} - epoch {epoch} - batch {i} - batch complete - loss {loss.item()}"
            )

        # the first rh call saves a json file with the loss from the worker at the end of the epoch
        rh.submit_result(
            f"logs/data_{worker_rank}_{epoch}.json",
            json.dumps(
                {
                    "loss": loss.item(),
                    "time": datetime.datetime.now().isoformat(),
                    "epoch": epoch,
                    "worker": worker_rank,
                }
            ),
        )
        # this saves the model. We only need to do it for one worker (so we picked worker 0)
        if worker_rank == 0:
            rh.submit_result("model.pkl", pickle.dumps(model.state_dict()))
```

To actually run the training job, first we spin up a Dask cluster and create a results handler object to manage the PyTorch results.


```python
n_workers = 3
cluster = SaturnCluster(n_workers=n_workers)
client = Client(cluster)
client.wait_for_workers(n_workers)

key = uuid.uuid4().hex
rh = results.DaskResultsHandler(key)
```

The next block of code starts the training job on all the workers, then uses the results handler to listen for results. The `process_results` function will hold the Jupyter notebook until the training job is done.


```python
futures = dispatch.run(client, train)
rh.process_results("/home/jovyan/project/training/", futures, raise_errors=False)
```

Lastly, we close the Dask workers


```python
client.close()
```

### Generating Names

To generate names, we have a function that takes the model and runs it over an over on a string generating each new character until a stop character is met.


```python
def generate_name(model, characters, str_len):
    in_progress_name = []
    next_letter = ""
    while not next_letter == "+" and len(in_progress_name) < 30:
        # prep the data to run in the model again
        in_progress_name_padded = in_progress_name[-str_len:]
        in_progress_name_padded = (
            list((str_len - len(in_progress_name_padded)) * "*") + in_progress_name_padded
        )
        in_progress_name_numeric = [characters.index(char) for char in in_progress_name_padded]
        in_progress_name_tensor = torch.tensor(in_progress_name_numeric)
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

To use the function we first need to load the model data from the training folder. That saved model state will be inserted into a parallel cuda model.


```python
# load the model and the trained parameters
model_state = pickle.load(open("/home/jovyan/project/training/model.pkl", "rb"))
model = torch.nn.DataParallel(Model()).cuda()
model.load_state_dict(model_state)
```

Finally lets generate 50 names! Also let's remove any names that would have shown up in the training data since those are less fun.


```python
# Generate 50 names then filter out existing ones
generated_names = [generate_name(model, characters, str_len) for i in range(0, 50)]
generated_names = [name for name in generated_names if name not in pet_names]
print(generated_names)
```

After running the code above you should see a list of names like:

```python
['Moicu', 'Caspa', 'Penke', 'Lare', 'Otlnys', 'Zexto', 'Toba', 'Siralto',
'Luny', 'Lit', 'Bonhe', 'Mashs', 'Riys Wargen', 'Roli', 'Sape', 'Anhyyhe',
'Lorla', 'Boupir', 'Zicka', 'Muktse', 'Musko', 'Mosdin', 'Yapfe', 'Snevi',
'Zedy', 'Cedi', 'Wivagok Rayten', 'Luzia', 'Teclyn', 'Pibty', 'Cheynet', 
'Lazyh', 'Ragopes', 'Bitt', 'Bemmen', 'Duuxy', 'Graggie', 'Rari', 'Kisi',
'Lvanxoeber', 'Bonu','Masnen', 'Isphofke', 'Myai', 'Shur', 'Lani', 'Ructli',
'Folsy', 'Icthobewlels', 'Kuet Roter']
```

## Conclusion

We've now successfully trained a PyTorch neural network on a distributed set of computers with Dask, and then used it to do NLP inference! Note that depending on the size of your data, your network architecture, and other parameters particular to your situation, training over a distributed set of machines may provide different amounts of a speed benefit. For an analysis of how much this can help, see our [blog post](https://saturncloud.io/blog/dask-with-gpus/) on training a neural network with multiple GPUs and Dask.
