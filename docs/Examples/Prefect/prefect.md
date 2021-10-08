# Develop a Scheduled Data Pipeline

Prefect is an open source workflow orchestration tool made for data-intensive workloads. In that project's language, you chain together a series of "tasks" into a "flow". If you're feeling confused, or would like an introduction to the concepts behind job scheduling with Prefect, you can start with our [Introduction to Prefect Concepts](<docs/Examples/Prefect/prefect_concepts.md>).


> This tutorial does not cover use of the Prefect Cloud hosted service. If you'd like to use that, [visit our tutorial on that service.](<docs/Examples/Prefect/prefect_cloud.md>)

## Dependencies

The [default images available with Saturn Cloud](<docs/Using Saturn Cloud/images.md>) contain everything you need to get started with Prefect.  If you are building your own images, make sure to include the following libraries:

- dask-saturn
- prefect
- dask
- bokeh

## Prefect Flow Example

The code described in this article is also available at https://github.com/saturncloud/examples/blob/main/examples/prefect/01-prefect.ipynb. That notebook contains sample code to take a Prefect flow and distribute its work with a Dask cluster.

### Define Tasks

The goal of this sample flow is to evaluate, on an ongoing basis, the performance of a model that predicts time-to-close for tickets in an IT support system.

That can be broken down into the following tasks:

- `get_trial_id()`: assign a unique ID to each run
- `get_ticket_data_batch()`: get a random set of newly-closed tickets
- `get_target()`: given a batch of tickets, compute how long it took to close them
- `predict()`: predict the time-to-close, using the heuristic "higher-priority tickets close faster"
- `evaluate_model()`: compute evaluation metrics comparing predicted and actual time-to-close
- `get_trial_summary()`: collect all evaluation metrics in one object
- `write_trial_summary()`: write trial results somewhere


```python

import json
import pandas as pd
import prefect
import numpy as np
import requests
import uuid

from datetime import datetime
from io import BytesIO
from prefect import task
from sklearn.metrics import mean_absolute_error
from sklearn.metrics import median_absolute_error
from sklearn.metrics import mean_squared_error
from sklearn.metrics import r2_score
from zipfile import ZipFile

from dask_saturn import SaturnCluster

@task
def get_trial_id() -> str:
    """
    Generate a unique identifier for this trial.
    """
    return str(uuid.uuid4())


@task
def get_ticket_data_batch(batch_size: int) -> pd.DataFrame:
    """
    Simulate the experience of getting a random sample of new tickets
    from an IT system, to test the performance of a model.
    """
    url = "https://archive.ics.uci.edu/ml/machine-learning-databases/00498/incident_event_log.zip"
    resp = requests.get(url)
    zipfile = ZipFile(BytesIO(resp.content))
    data_file = "incident_event_log.csv"
    # _date_parser has to be a lambda or pandas won't convert dates correctly
    _date_parser = lambda x: pd.NaT if x == '?' else datetime.strptime(x, "%d/%m/%Y %H:%M")
    df = pd.read_csv(
        zipfile.open(data_file),
        parse_dates=[
            "opened_at",
            "resolved_at",
            "closed_at",
            "sys_created_at",
            "sys_updated_at"
        ],
        infer_datetime_format=False,
        converters={
            "opened_at": _date_parser,
            "resolved_at": _date_parser,
            "closed_at": _date_parser,
            "sys_created_at": _date_parser,
            "sys_updated_at": _date_parser
        },
        na_values = ['?']
    )
    df["sys_updated_at"] = pd.to_datetime(df["sys_updated_at"])
    rows_to_score = np.random.randint(0, df.shape[0], 100)
    return(df.iloc[rows_to_score])


@task
def get_target(df):
    """
    Compute time-til-close on a data frame of tickets
    """
    time_til_close = (df['closed_at'] - df['sys_updated_at']) / np.timedelta64(1, 's')
    return time_til_close


@task
def predict(df):
    """
    Given an input data frame, predict how long it will be until the ticket is closed.
    For simplicity, using a super simple model that just says
    "high-priority tickets get closed faster".
    """
    seconds_in_an_hour = 60.0 * 60.0
    preds = df["priority"].map({
        "1 - Critical":   6.0 * seconds_in_an_hour,
        "2 - High":      24.0 * seconds_in_an_hour,
        "3 - Moderate": 120.0 * seconds_in_an_hour,
        "4 - Lower":    240.0 * seconds_in_an_hour,
    })
    default_guess_for_no_priority = 180.0 * seconds_in_an_hour
    preds = preds.fillna(default_guess_for_no_priority)
    return(preds)


@task
def evaluate_model(y_true, y_pred, metric_name: str) -> float:
    metric_func_lookup = {
        "mae": mean_absolute_error,
        "medae": median_absolute_error,
        "mse": mean_squared_error,
        "r2": r2_score
    }
    metric_func = metric_func_lookup[metric_name]
    return metric_func(y_true, y_pred)


@task
def get_trial_summary(trial_id:str, actuals, input_df: pd.DataFrame, metrics: dict) -> dict:
    out = {"id": trial_id}
    out["data"] = {
        "num_obs": input_df.shape[0],
        "metrics": metrics,
        "target": {
            "mean": actuals.mean(),
            "median": actuals.median(),
            "min": actuals.min(),
            "max": actuals.max()
        }
    }
    return out


@task(log_stdout=True)
def write_trial_summary(trial_summary: str):
    """
    Write out a summary of the file. Currently just logs back to the
    Prefect logger
    """
    logger = prefect.context.get("logger")
    logger.info(json.dumps(trial_summary))
```

### Construct a Flow

Now that all of the task logic has been defined, the next step is to compose those tasks into a "flow". 

Because we want this job to run on a schedule, the code below provides one additional argument to Flow(), a special "schedule" object. In this case, the code below says "run this flow every minute". For more on schedules, see [the Prefect docs](https://docs.prefect.io/core/concepts/schedules.html).

> Prefect flows do not have to be run on a schedule. To test a single run, just omit `schedule` from the code block below.

```python

from datetime import timedelta
from prefect import Flow
from prefect.schedules import IntervalSchedule

schedule = IntervalSchedule(
    interval=timedelta(minutes=1)
)
with Flow('ticket-model-evaluation', schedule) as flow:
    batch_size = Parameter(
        'batch-size',
        default=1000
    )
    trial_id = get_trial_id()

    # pull sample data
    sample_ticket_df = get_ticket_data_batch(batch_size)

    # compute target
    actuals = get_target(sample_ticket_df)

    # get prediction
    preds = predict(sample_ticket_df)

    # compute evaluation metrics
    mae = evaluate_model(actuals, preds, "mae")
    medae = evaluate_model(actuals, preds, "medae")
    mse = evaluate_model(actuals, preds, "mse")
    r2 = evaluate_model(actuals, preds, "r2")

    # get trial summary in a string
    trial_summary = get_trial_summary(
        trial_id=trial_id,
        input_df=sample_ticket_df,
        actuals=actuals,
        metrics={
            "MAE": mae,
            "MedAE": medae,
            "MSE": mse,
            "R2": r2
        }
    )

    # store trial summary
    trial_complete = write_trial_summary(trial_summary)
````

If you execute

```python
flow.run()
```

The flow will execute locally.  It's a good idea to start locally, with a smaller subset of your data before scaling up.

***

## Scaling up with Dask Clusters
Scaling up really just requires making some changes to the `flow.run()` call, as you can see here.

### Dask LocalCluster

A Dask Local cluster is good for taking advantage of all the cores on your machine.  If you don't need a huge amount of resources to successfully complete the job, spinning up a large instance and using Dask local cluster is a great idea.

```python
from prefect.engine.executors import DaskExecutor
flow.run(executor=DaskExecutor())
```

### Dask Multi-Node Cluster
```python
from dask_saturn import SaturnCluster
from prefect.engine.executors import DaskExecutor

flow.run(
    executor=DaskExecutor(
        cluster_class=SaturnCluster,
        cluster_kwargs=dict(
            scheduler_class="xlarge",
            worker_class="16xlarge",
            nprocs=16,
            nthreads=4,
            n_workers=5,
            autoclose=True
        )
    )
)

```

This tells Prefect to create a `SaturnCluster` when it starts the flow, and stop it (`autoclose=True`) when it's complete.
