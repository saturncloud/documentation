# Glossary of Terms

* **API:** "Application Programming Interface". Interface for developers or coders to interact with a system, library, or framework. It provides developers with standard commands for performing common operations so they do not have to write the code from scratch.  
* **AWS**: Amazon Web Services, a cloud computing platform offered by Amazon. Offers a wide range of products similar to GCP or Azure, including data storage, on-demand compute resources, and Virtual Private Cloud. 
* **Azure**: Microsoft Azure, a cloud computing platform offered by Microsoft. Offers a wide range of products similar to GCP or AWS, including data storage, on-demand compute resources, and Virtual Private Cloud. 
* **Cluster:** A group of computers that work together that perform a similar function or collaborate on the same function.
* **CPU:** "Central Processing Unit". A component within a computer that receives instructions and follows them to process data or information. Contains at least one core (ALU), control unit, and memory cache.
* **Credential:** One of a variety of digital passcodes or methods of authentication to confirm the identity of a user or computer. Should be stored in secure formats, not plain text. 
* **CUDA:** "Compute Unified Device Architecture", though the acronym is the only term used today. A parallel computing platform and application programming interface (API) model created by NVIDIA. The GPUs usually used in Saturn Cloud applications have NVIDIA processors with the CUDA framework.
* **Dashboard:** A user interface displaying visualizations and/or data with realtime or near realtime updating. May be created and "served" by a number of different tools, including Python libraries Voila and Panel. [More information](/docs).
* **Dask**: A parallel computation framework for Python. Go to [Dask.org](https://dask.org/) for more information.
* **Dask Cluster**: A group of computers that work together to run workloads using Dask. Enables parallelization across multiple machines.
* **Dask Worker:** One computer in a Dask Cluster, specifically assigned to complete calculations.
* **Dask Scheduler:** One computer in a Dask Cluster, specifically assigned to pass instructions to Dask Workers. Usually only one per cluster is needed.
* **Delayed Function:** A function designed to be defined at one point, but not actually run until a later time. See also "Lazy Evaluation".
* **Deployment:** A Saturn Cloud resource that is used to host continuously running tasks like having a Dashboard or API. Deployments have a single command that starts running when the resource starts and stays running until the resource is turned off.
* **Eager Evaluation:** Opposite of Lazy Evaluation. The name for a strategy where functions are defined, and run instantly at the time of defining.
* **EC2**: EC2 is the shorthand for Amazon Web Services (AWS) Elastic Cloud Computing service. Through this service, users can rent computation resources for a period of time, and pay for use for only as long as they need it. Our customers use these instances to get compute resources on Saturn Cloud.  https://aws.amazon.com/ec2/  
* **GCP**: Google Cloud Platform, also known as just Google Cloud. A cloud computing platform offered by Google. Offers a wide range of products similar to AWS or Azure, including data storage, on-demand compute resources, and Virtual Private Cloud. 
* **Git:** Software for tracking changes in files, usually code. Allows users to easily share code and see the differences between versions of one file. Commercial tools where users can access git include Github, Gitlab, and Bitbucket.
* **GPU:** "Graphics Processing Unit". Alternative to CPU. A computer processor optimized for rendering graphics, but also very effective for highly parallel machine learning code. 
* **IDE**: "Integrated Development Environment". An application allowing writing, running, and debugging code in a unified product. Some Python users use IDEs such as VSCode, PyCharm, or Sublime.
* **Job:** A type of Saturn Cloud resource used to run a task either at will or on a schedule. The job resource has a command that is executed on the resource when the job is run.
* **JupyterLab:** A full featured user interface for interactive computation in Julia, Python, or R. It offers multiple tools besides the Jupyter Notebook, including terminal, text editor, file browser, visual widgets, rich outputs, etc. 
* **Jupyter Notebook**: An interactive computational environment, designed specifically for creating and running interactive code in Julia, Python, or R languages. 
* **Jupyter Server**: A server that hosts a Jupyter process, which runs *JupyterLab* (or optionally a *Jupyter Notebook*) inside the Saturn Cloud product.
* **Lazy Evaluation:** The name for a strategy where functions are defined at one point, but not actually run until a later time. See also "Delayed Function".
* **Local Cluster**: A collection of processes on a *single* computer that run workloads using Dask. Enables parallelization on one machine.
* **Notebook:** An interactive computational environment. For Saturn Cloud, this usually refers to Jupyter Notebook, which is designed specifically for creating and running interactive code in Julia, Python, or R languages. 
* **Parallelization, Parallel Computing:** Computation where many calculations or the execution of processes are carried out simultaneously. May be conducted on one computer or clusters of many computers. In Saturn Cloud, Dask is the framework we use to enable parallelization.
* **Prefect:** An open source workflow orchestration tool made for data-intensive workloads. This allows you to schedule and organize jobs to run any time, in your chosen order. It can accommodate dependencies between jobs, and is very useful for data pipelines and machine learning workflows. [More information](/docs).
* **Resource:** In Saturn Cloud, a resource a distinct computing environment with an optionally attached Dask cluster. Resources are the core items of Saturn Cloud and users will likely have different resources for different projects or types of work. More information on creating a resource can be found on the [Starting a Resource page](<docs/Getting Started/start_resource.md>). Resources can one of several types:
  * **Jupyter Server** - The most common type of resource, these are used for exploratory analysis and data science daily work.
  * **Job** - For one off tasks that happen at will or on a schedule.
  * **Deployment** - For continuously running tasks like hosting a dashboard or API.
  * **Prefect Cloud Flow** - For tasks that use Prefect Cloud to schedule, run, and log.
* **S3**: A part of the AWS product suite, also known as Amazon Simple Storage Service. This allows large scale storage of many different types of data, including relatively unstructured data. Many of our users store data in S3 and load it to Saturn Cloud for analysis.   
* **Scheduler**: One computer in a computing cluster, specifically assigned to pass instructions to Workers. Usually only one per cluster is needed.
* **Snowflake**: A company offering SQL based cloud data storage and analysis products. Saturn Cloud offers robust integration with Snowflake data storage solutions. <a href="https://www.snowflake.com/" target='_blank' rel='noopener'>Learn more at their website</a>.
* **Spot Instance**: A Spot Instance is a type of AWS EC2 machine that you can request. It is heavily discounted off the standard price, but may not always be available as it depends on where excess capacity exists. <a href="https://aws.amazon.com/ec2/spot/" target='_blank' rel='noopener'>Learn about EC2 Spot Instances on AWS website</a>.
* **vCPU**: Virtual CPU. The equivalent computing power/resources of a CPU, but distributed across multiple actual pieces of hardware. Allows developers to use the computing resources in time slots, sharing with other users. A vCPU represents the computing power of a CPU, across multiple resources. 
* **VPC**: Virtual Private Cloud. A pool of shared resources allocated within a public cloud environment, providing privacy or isolation between different users or organizations. Resources are in the cloud, but access is restricted to authorized parties and data is not shared between users.
* **Weights & Biases**: A cloud based product allowing data scientists to easily and powerfully monitor the performance of model training and inference. Saturn Cloud offers robust integration with Weights & Biases solutions. <a href="https://wandb.ai/" target='_blank' rel='noopener'>Learn more at their website</a>.
* **Worker:** One computer in a computing cluster, specifically assigned to complete calculations.
