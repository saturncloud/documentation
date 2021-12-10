# Prefect Concepts


Prefect is an open source workflow orchestration tool made for data-intensive workloads. This  allows you to schedule and organize jobs to run any time, in your chosen order. It can accommodate dependencies between jobs, and is very useful for data pipelines and machine learning workflows.

Saturn Cloud supports two different tools in the Prefect ecosystem: 
* **Prefect Core**, which is an open source library users can install, and 
* **Prefect Cloud**, which is a fully hosted cloud service. The same team that maintains the `prefect` core library runs Prefect Cloud. Prefect Cloud is a hosted, high-availability, fault-tolerant service that handles all the orchestration responsibilities for running data pipelines.


## Prefect Core

Prefect is a Python-based workflow management system which helps in building , running and monitoring millions of data workflow and pipelines. It helps in improving behavior of your workflows and handling missing engineering. This means that Prefect will ensure that either your code is successfully executed or if things go south, it fails successfully. 

Let’s say our goal is to clean a user activity data for a music website. A simple workflow is :\
fetch data -> extract columns -> create new features -> handle noise ->save 

This workflow looks fine , but only till things don’t go wrong . What happens when fetching data fails or there occurs an unexpected issue while creating new features . This is where prefect comes into use . Prefect makes it easy to add precautions to your workflow , improving its overall behavior. 



> For a more in-depth discussion of these concepts, visit the <a href="https://docs.prefect.io/core/" target='_blank' rel='noopener'>Prefect Core documentation</a>. They definitely have the best information about their tools!
If you are familiar with <a href="https://airflow.apache.org/" target="_blank" rel="noopener">Apache Airflow</a>, you might best understand Prefect through comparisons to that project. See <a href="https://docs.prefect.io/core/getting_started/why-not-airflow.html" target="_blank" rel="noopener">"Why not Airflow"</a> in the Prefect documentation for more.

### Prefect Core Basics
## Tasks
A Task represents a discrete action in a Prefect workflow. This is easy to conceptualize as a function - it's a standalone chunk of work to be done. This chunk of work might accept inputs and it might produce outputs.

### An Example Task
```python
from prefect import task
@task
def add(x, y=1):
    return x + y
```
## Flows
A "flow" is a container for multiple tasks which understands the relationship between those tasks. Tasks are arranged in a directed acyclic graph (DAG), which just means that there is always a notion of which tasks depend on which other tasks, and those dependencies are never circular.
When you are creating a Prefect flow, you're developing a set of tasks, then instructing Prefect on when they should be run, and in what order. Prefect also offers flexible control over what to do when a specific task or entire flow fails.
### An Example Flow
```python
from prefect import Flow, task
@task
def add(x, y=1):
    return x + y
with Flow("My first flow") as flow:
    first_result = add(1, y=2)
    second_result = add(x=first_result, y=100)
```
## Running a Flow
Running a flow is as simple as running `flow.run()` after your flow is defined. 

At this point, you have the basic concepts behind Prefect Core! Now, you might want to see some more complex examples, so go [visit our Prefect tutorials to learn more](<docs/Examples/Prefect/qs-01-prefect-singlenode.md>).