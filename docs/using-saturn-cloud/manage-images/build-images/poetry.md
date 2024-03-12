# Create Docker Images with Conda and Poetry

[Poetry](https://python-poetry.org/) is a common tool used for Python packaging and dependency management. [Conda](https://docs.conda.io/en/latest/) is a virtual environment system commonly used in data science. This article goes over how to build Saturn Cloud images with Poetry and Conda. This approach is pretty general, and can be used to build your own docker images even if you're not a Saturn Cloud user.

The Saturn Cloud image build tool understands conda and pip configuration. If your organization uses poetry, you can build docker images using poetry using your `pyproject.toml` file. To do so, enter the following into the `postBuild` section of the image build.

```
mkdir -p /tmp/poetry

tee -a /tmp/poetry/pyproject.toml > /dev/null <<EOT

[tool.poetry]
name = "my-package"
version = "0.1.0"
description = "The description of the package"
license = "MIT"
authors = [
    "Hugo Shi <hugo@saturncloud.io>",
]
[tool.poetry.dependencies]
# Compatible Python versions
python = ">=3.8"
flask = "2.2.2"

# Dependency groups are supported for organizing your dependencies
[tool.poetry.group.dev.dependencies]
pytest = "^7.1.2"

EOT

export POETRY_HOME=/opt/poetry
export PATH=/opt/poetry/bin:${PATH}
sudo mkdir -p /opt/poetry
sudo chown jovyan:jovyan -R /opt/poetry
curl -sSL https://install.python-poetry.org | python -
mamba install -n saturn python=3.9 pip ipykernel
export PATH=/opt/saturncloud/envs/saturn/bin:${PATH}
poetry config cache-dir /opt/poetry
poetry config virtualenvs.create false
poetry config virtualenvs.prefer-active-python true
export CONDA_PREFIX=/opt/saturncloud/envs/saturn
export CONDA_DEFAULT_ENV=saturn
cd /tmp/poetry && poetry install
/opt/saturncloud/envs/saturn/bin/python -m ipykernel install --name python3 --prefix=/opt/saturncloud
```

## How does this work?
### Configuration Files

```
mkdir -p /tmp/poetry

tee -a /tmp/poetry/pyproject.toml > /dev/null <<EOT

[tool.poetry]
name = "my-package"
version = "0.1.0"
description = "The description of the package"
license = "MIT"
authors = [
    "Hugo Shi <hugo@saturncloud.io>",
]
[tool.poetry.dependencies]
# Compatible Python versions
python = ">=3.8"
flask = "2.2.2"

# Dependency groups are supported for organizing your dependencies
[tool.poetry.group.dev.dependencies]
pytest = "^7.1.2"

EOT
```

The above snippet dumps your pyproject.toml file into `/tmp/poetry`. If you are building images outside of Saturn Cloud and would prefer to do this directly in the Dockerfile, you can also do:

```
COPY pyproject.toml /tmp/poetry/pyproject.toml
```

### Installing Poetry

```
export POETRY_HOME=/opt/poetry
export PATH=/opt/poetry/bin:${PATH}
sudo mkdir -p /opt/poetry
sudo chown jovyan:jovyan -R /opt/poetry
curl -sSL https://install.python-poetry.org | python -
```

The above creates a `POETRY_HOME` directory, and installs poetry into it. The Saturn Cloud image builder does not allow you to set ENV vars in the docker image. To do so, you could add the following to your resource start scripts:

```
export POETRY_HOME=/opt/poetry
export PATH=/opt/poetry/bin:${PATH}
```

If you are building docker images outside of Saturn Cloud, you could do:

```
ENV POETRY_HOME=/opt/poetry
ENV PATH=/opt/poetry/bin:${PATH}
```

### Setting up the conda environment with the ipykernel

```
mamba install -n saturn python=3.9 pip ipykernel
export PATH=/opt/saturncloud/envs/saturn/bin:${PATH}
export CONDA_PREFIX=/opt/saturncloud/envs/saturn
export CONDA_DEFAULT_ENV=saturn
```

The above installs `python`, `pip`, and `ipykernel` into a conda env. `ipykernel`. ipykernel is only necessary to use this env with Jupyter. Poetry tries to detect the currently activated conda env, to figure out which python to use. setting `CONDA_PREFIX` and `CONDA_DEFAULT_ENV` helps poetry figure out which Python environment we want.


### Instaling packages from pyproject.toml

```
poetry config cache-dir /opt/poetry
poetry config virtualenvs.create false
poetry config virtualenvs.prefer-active-python true
cd /tmp/poetry && poetry install
/opt/saturncloud/envs/saturn/bin/python -m ipykernel install --name python3 --prefix=/opt/saturncloud
```
Here we configure poetry so that it does not create a virtual env (since we want to use the existing conda env). The `poetry install` command should install python packages into the activated conda environment.

{{% images_docs_view %}}
