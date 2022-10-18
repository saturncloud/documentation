# Comet


## Overview

[Comet](https://www.comet.ml/) is a platform for tracking, comparing, and visualizing your modeling workflows. This example shows how to use Comet on the Saturn Cloud platform by creating a PyTorch classification model of the MNIST digits dataset, but Comet can be used to log many types of model training. Check out the rest of the [Comet documentation](https://www.comet.ml/docs/) for examples of how to integrate Comet with other frameworks and languages. 

## Process

### Imports

To properly log your experiments, run `import comet_ml` at the beginning of your script. This line needs to run first for Comet to log properly.


```python
import comet_ml
```

After you import `comet_ml`, import the rest of your libraries as you would normally.


```python
import torch
import torch.nn as nn
import torchvision.datasets as dsets
import torchvision.transforms as transforms
from torch.autograd import Variable
```

### Find your Comet API key

Go to your [Comet settings page](https://www.comet.ml/api/my/settings) and copy your API key.

### Link your API key and define your experiment

The core class of Comet is an Experiment. An Experiment allows you to log your model hyperparameters, code, metrics and model configuration.  The `Experiment` class requires you to specify your Comet credentials to properly authenticate your session. 

You can specify the API key in several ways:
1. Within your notebook by setting an environment variable called "COMET_API_KEY".
2. Within the Saturn Cloud [secrets manager](https://saturncloud.io/docs/using-saturn-cloud/credentials/) by adding an environment variable named "COMET_API_KEY". 
    > This method is more secure and allows you to use your key over multiple Saturn Cloud resources.
3. By following the [Comet instructions](https://www.comet.ml/docs/python-sdk/advanced/#non-interactive-setup) to create a `.comet.config` file for use in this resource.

Once you have specified your credentials using your preferred method, run the following cell to define your Experiment. The function will output a link to comet.ml where you can view the modeling logs.


```python
# import os
# os.environ["COMET_API_KEY"] = "YOUR-API-KEY"

experiment = comet_ml.Experiment(project_name="pytorch")
```

Next, we specify and log the hyperparameters for this session as a dictionary.


```python
hyper_params = {
    "sequence_length": 28,
    "input_size": 28,
    "hidden_size": 128,
    "num_layers": 2,
    "num_classes": 10,
    "batch_size": 100,
    "num_epochs": 2,
    "learning_rate": 0.01,
}
experiment.log_parameters(hyper_params)
```

### Set up the modeling run

The next cells are exactly the same as you would expect in any other PyTorch workflow. Load the MNIST dataset from `torchvison`, specify your dataloaders, define the model based on your hyperparameters, and define your loss and optimizer functions.

#### Load the MNIST dataset


```python
train_dataset = dsets.MNIST(
    root="./data/", train=True, transform=transforms.ToTensor(), download=True
)

test_dataset = dsets.MNIST(root="./data/", train=False, transform=transforms.ToTensor())
```

#### Create the data loaders


```python
# Data Loader (Input Pipeline)
train_loader = torch.utils.data.DataLoader(
    dataset=train_dataset, batch_size=hyper_params["batch_size"], shuffle=True
)

test_loader = torch.utils.data.DataLoader(
    dataset=test_dataset, batch_size=hyper_params["batch_size"], shuffle=False
)
```

#### Specify the RNN model (many-to-one)


```python
class RNN(nn.Module):
    def __init__(self, input_size, hidden_size, num_layers, num_classes):
        super(RNN, self).__init__()
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        self.lstm = nn.LSTM(input_size, hidden_size, num_layers, batch_first=True)
        self.fc = nn.Linear(hidden_size, num_classes)

    def forward(self, x):
        # Set initial states
        h0 = Variable(torch.zeros(self.num_layers, x.size(0), self.hidden_size))
        c0 = Variable(torch.zeros(self.num_layers, x.size(0), self.hidden_size))

        # Forward propagate RNN
        out, _ = self.lstm(x, (h0, c0))

        # Decode hidden state of last time step
        out = self.fc(out[:, -1, :])
        return out


rnn = RNN(
    hyper_params["input_size"],
    hyper_params["hidden_size"],
    hyper_params["num_layers"],
    hyper_params["num_classes"],
)
```

#### Define the loss function and optimizer


```python
criterion = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(rnn.parameters(), lr=hyper_params["learning_rate"])
```

### Train the model

Finally, we train the model. There are a few additions here for additional logging in Comet. The first is the `with experiment.train()` and `with experiment.test()`, which tells Comet that the following code is part of a training or test workflow respectively and should be logged as such. We also log our metric of choice using `experiment.log_metric()`.


```python
# Train the Model
with experiment.train():
    step = 0
    for epoch in range(hyper_params["num_epochs"]):
        correct = 0
        total = 0
        for i, (images, labels) in enumerate(train_loader):
            images = Variable(
                images.view(-1, hyper_params["sequence_length"], hyper_params["input_size"])
            )
            labels = Variable(labels)

            # Forward + Backward + Optimize
            optimizer.zero_grad()
            outputs = rnn(images)
            loss = criterion(outputs, labels)
            loss.backward()
            optimizer.step()

            # Compute train accuracy
            _, predicted = torch.max(outputs.data, 1)
            batch_total = labels.size(0)
            total += batch_total

            batch_correct = (predicted == labels.data).sum()
            correct += batch_correct

            # Log batch_accuracy to Comet.ml; step is each batch
            step += 1
            experiment.log_metric("batch_accuracy", batch_correct / batch_total, step=step)

            if (i + 1) % 100 == 0:
                print(
                    "Epoch [%d/%d], Step [%d/%d], Loss: %.4f"
                    % (
                        epoch + 1,
                        hyper_params["num_epochs"],
                        i + 1,
                        len(train_dataset) // hyper_params["batch_size"],
                        loss.item(),
                    )
                )

        # Log epoch accuracy to Comet.ml; step is each epoch
        experiment.log_metric("batch_accuracy", correct / total, step=epoch)


with experiment.test():
    # Test the Model
    correct = 0
    total = 0
    for images, labels in test_loader:
        images = Variable(
            images.view(-1, hyper_params["sequence_length"], hyper_params["input_size"])
        )
        outputs = rnn(images)
        _, predicted = torch.max(outputs.data, 1)
        total += labels.size(0)
        correct += (predicted == labels).sum()

    experiment.log_metric("accuracy", correct / total)
    print("Test Accuracy of the model on the 10000 test images: %d %%" % (100 * correct / total))
```

### Complete your logging session

Lastly, because we are working in a Jupyter Notebook, we need to call `experiment.end()`. This will fully sync the run with Comet to complete the logging. If you are running your training code as a script, you do not need this line.


```python
experiment.end()
```

Your training run will now be shown on your Comet dashboard under a project called "pytorch". Notice that Comet logged the full environment specifications, model metrics, code, and more. Try running the code again with different hyperparameters to see and compare the additional training runs.
