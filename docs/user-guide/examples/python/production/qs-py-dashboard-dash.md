# Create a Dashboard with Dash


## Overview
With [Dash](https://dash.plotly.com/) from Plotly, data scientists can produce low-code data apps by abstracting away much of the technologies and protocols typically required for interactive data visualization. Deploying a Dash app on Saturn Cloud provides not only a scalable backend for your app but also a url link for dissemination.

In this example, we create a simple UI showing a [Uniform Manifold Approximation and Projection (UMAP)](https://umap-learn.readthedocs.io/en/latest/) model projection of the famous MNIST digits and fashion datasets. The app will read the data, train the UMAP model, and produce a 3D graph of the result using [plotly](https://plotly.com/python/).

## Creating the App
All the app code is contained in a file called "app.py." To deploy this app on Saturn Cloud, call `python app.py` as the command in a Saturn Cloud deployment. See [Saturn Cloud docs](https://saturncloud.io/docs/user-guide/examples/dashboards/dashboard/) for more detailed instructions on deploying this and other dashboards.

The "app.py" file contains several sections to create the Dash app.

### Import the Libraries

This exercise uses Dash, plotly, and UMAP to create a dashboard app.

``` python
import numpy as np
import pandas as pd
import plotly.express as px
from dash import Dash, Input, Output, dcc, html
from umap import UMAP
```
### Define the App and Layout

Next, define the app, then specify the layout. Some of this code might seem familiar if you work with HTML. Dash uses functions like `html.div` to define html components.

You can use Dash Core Components (`dcc`) such as `Dropdown` and `Graph` to define user input and output components.

This example creates two columns. The first contains a dropdown for the user to select a dataset, and the second a visualization of the UMAP projection.

See the [Dash documentation](https://dash.plotly.com/) for more information about defining specific layouts.

``` python
app = Dash(__name__)
app.title = "UMAP Projections"

app.layout = html.Div(
    [
        html.Div(
            children=[
                html.H1(
                    "UMAP Projections for MNIST and Fashion-MNIST Datasets",
                    style={"text-align": "center"},
                ),
                dcc.Markdown(
                    """
                    Uniform Manifold Approximation and Projection (UMAP) is a general-purpose dimension reduction algorithm. Similar to t-distributed stochastic neighbor embedding (t-SNE), you can use UMAP to visualize the relationships between data points. In this example, we are training a three-component UMAP model on MNIST datasets and then displaying the 3D graph of the result. The color of the point in the graph is based on the label. In the resulting graph, blobs of colors show that UMAP clustered data points with similar labels together.
                """,
                ),
            ],
            style={"padding": 10},
        ),
        html.Div(
            [
                html.Div(
                    children=[
                        html.H1("Input"),
                        html.Label("Dataset"),
                        dcc.Dropdown(
                            ["MNIST-Digits", "MNIST-Fashion"], "MNIST-Digits", id="dataset_dropdown"
                        ),
                    ],
                    style={"padding": 10, "flex": 1},
                ),
                html.Div(
                    children=[
                        html.H1("Output"),
                        dcc.Loading(
                            id="loading-1",
                            children=[dcc.Graph(id="graph")],
                            type="circle",
                        ),
                    ],
                    style={"padding": 10, "flex": 3},
                ),
            ],
            style={"display": "flex", "flex-direction": "row"},
        ),
    ]
)
```

### Define the Callbacks
Dash uses callbacks to run functions based on user input. For this example, we define a single callback to update the graph when the dataset changes.

Dash uses `Input` and `Output` to define callback inputs and outputs. The first parameter is the id of the associated component in the layout, and the second is the type of information passed to and from the function.

For this example, the callback downloads the correct dataset based on the dropdown selection, runs `fit_transform`