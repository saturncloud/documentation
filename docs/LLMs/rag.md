# Creating a RAG pipeline with LangChain
Retrieval-augmented generation (RAG) enables large language models (LLMs) to access extensive knowledge bases, querying them to retrieve relevant information and generate more accurate, well-informed responses.

RAG enables LLMs to access up-to-date information in real-time, even if the models weren't initially trained on the specific subject, allowing them to provide more insightful answers. However, this added flexibility brings increased complexity in designing effective RAG pipelines. To simplify these challenges, Langchain provides an efficient and streamlined solution for setting up prompting pipelines.

## Getting Started

In this article, we'll show you how to:

- Create a Saturn Cloud development machine where you will:
  - Create a vector database with Pinecone.
  - Add custom data to your vector database via LangChain.
  - Attach an LLM(with OpenAI or your own self hosted model) to your vector database.
- create a Saturn Cloud deployment to serve this pipeline with MLFlow.

## Create a development machine

Please click on "Python Server", select a GPU instance type, and select the `saturn-python-llm` image, version `2024.08.01`. This image will have all the dependencies required to run this code. Once the workspace has been created, please click on the "Manage" tab. "Edit" the API token, and un-check the "scope" toggle button. Saturn Cloud API tokens default to operating only on the current resource. Because we will want to interact with other resources (like the RAG pipeline we are about to deploy) we remove the token scope which gives the token complete access to act on your behalf. You can [learn more about API tokens here](/docs).

### API Keys for external services.
Before we begin, please sign up for pinecone and obtain a `PINECONE_API_KEY`. If you plan on deploying RAG with one of OpenAIs models, make sure you have an `OPENAI_API_KEY`. If you plan on hosting your own LLM, you may need to obtain and configure a `HF_TOKEN` depending on whether or not your model is private.

To [learn how to insert these secrets, click here](/docs).

## The code

### Import libraries for vector stores, chat models, and dataset handling

```
import os
import mlflow
import mlflow.pyfunc
import pinecone as pc
from langchain_core.documents import Document
from langchain_pinecone import PineconeEmbeddings, PineconeVectorStore
from langchain_openai import ChatOpenAI
from langchain_community.llms.vllm import VLLMOpenAI
from langchain.chains import RetrievalQAWithSourcesChain
from datasets import load_dataset
from tqdm import tqdm
```

### Step 1: Load the data

In our example, we'll download a hosted cardiology dataset from PubMed. This dataset is a collection of PubMed abstracts from journals related to cardiology. The dataset has already been chunked into smaller pieces for consumption by a vector store.

```
dataset = load_dataset("saturncloud/pubmed-cardiology")["train"]
```

### Step 2: Initialize Pinecone

Now that we have our dataset loaded, our aim in this step is to create a vector database using Pinecone. In the following step, we'll put this dataset into Pinecone.

Before we insert our dataset into Pinecone, we'll need to define the following:

- index_name: The name we want to call our vector index on Pinecone.
- model_name: This is a retrieval model from HuggingFace used for retrieval, and for creating embeddings of our data to be stored into Pinecone.
- cloud and region: Defining a cloud provider and instance location.
- pinecone: Our initialized Pinecone client.
- spec: Our serverless instance specifications.
- index_name = "pubmed"
- model_name = "multilingual-e5-large"

```
cloud = os.environ.get('PINECONE_CLOUD') or 'aws'
region = os.environ.get('PINECONE_REGION') or 'us-west-2'

pinecone = pc.Pinecone(api_key=os.environ.get("PINECONE_API_KEY"))
spec = pc.ServerlessSpec(cloud=cloud, region=region)
```

Once we have our Pinecone client, we can use this client to create a new vector index with create_index().

```
# Create the index if it doesn't exist

if not index_name in [x['name'] for x in pinecone.list_indexes()]:
    pinecone.create_index(index_name, dimension=1024, metric='cosine', spec=spec)

    # Wait for the vector index to be fully initialized.
    while not pinecone.describe_index(index_name).status['ready']:
        time.sleep(1)
    print("Vector Index Initialized.")

else:
    print("Index", index_name, "already exists.")
```

Now, we'll initialize PineconeVectorStore (a LangChain vector store object), allowing us to easily pass our dataset through an embedding model to create embeddings to be stored in our vector database (index). Once we have all of our Pinecone components defined and initialized, it's time for us to move on to parsing our dataset and feeding the dataset into Pinecone.

```
index = pinecone.Index(index_name)
vector_store = PineconeVectorStore(
    index=index,
    embedding=PineconeEmbeddings(model=model_name, pinecone_api_key=os.environ.get("PINECONE_API_KEY"))
)
```

### Step 3: Adding Data to Pinecone
Once we have Pinecone initialized, we want to add our data into our vector index.

Using LangChain, we'll pick data that is related to cholesterol. This is done just to make the data insert process faster. In our loop, we extract the main content and all metadata fields to the text and meta variables respectively.

text and meta are then used to create a LangChain Document object, in which we append to the documents list for importing to Pinecone.

```
documents = []
for i in tqdm(range(len(dataset))):
    d = dataset[i]
    text = d.pop('text')
    if 'cholesterol' not in text:
        continue
    meta = {k: v for k, v in d.items() if v is not None}
    doc = Document(page_content=text, metadata=meta)
    documents.append(doc)
```

Once our cholesterol-related articles have been parsed to LangChain Documents, let's add these documents to our vector store.

For our vector store, we'll pass in our documents list with a corresponding id for each document ranging from 0 to the number of documents in our list ids.

Note: The uploading process takes some time (~20 minutes for our dataset). If you would like to view the upload progress, go to your Pinecone dashboard > Databases > Indexes, and view your record count.

```
ids = [str(x) for x in range(len(documents))]
vector_store.add_documents(documents=documents, ids=ids)
```

### e
Step 4: Retrieving Documents with LLMs
Next, we'll use the embeddings we've stored in Pinecone with an LLM. For simplicity, we'll use OpenAI's gpt-3.5-turbo.

If you would like to learn how to host your own models using LangChain, click here.

First, using LangChain's ChatOpenAI(), we'll initialize our llm instance.
```
llm = ChatOpenAI(
    openai_api_key=os.environ['OPENAI_API_KEY'],
    model_name='gpt-3.5-turbo',
    temperature=0.0
)
```

If you've already deployed your own language model, you can use the following snippet instead.

```
llm = VLLMOpenAI(
    openai_api_base="url-to-your-deployment"
    model_name='name-of-your-model',
)
```

Then, we'll use LangChain's `RetrievalQAWithSourcesChain()` to combine the LLM and a retriever (our vector_store). This allows the system to retrieve relevant documents using the retriever and use them to provide answers with sources attached.

```
qa_with_sources = RetrievalQAWithSourcesChain.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=vector_store.as_retriever()
)
```

Now, let's try to ask a question that is specific to the dataset we have inserted into Pinecone. To view these documents, simply browse through the text field in documents. If you would like to view these documents in the vector store, go to your "databases" tab in your Pinecone dashboard.

Here is text from a document we pulled from our vector index:

> Serum lipid levels of 10,977 normal Japanese subjects in 1980 were determined by a joint study of 14 institutions, specializing in lipid research, located in 9 districts of Japan. The data obtained were compared with those in 1960 and 1970. Total cholesterol (TC) levels in 1980 increased with age except for the 1st decade and reached maximum (205 mg/dl) at the 7th decade. The mean value in any age was higher than that of 20 years ago by 10-15 mg/dl. Triglyceride (TG) levels also increased with age and reached maximum (130 mg/dl) at the 7th decade. The mean values of subjects over the 5th decade were higher than those of 10 years ago by 10-20 mg/dl. In contrast with TC and TG, HDL-cholesterol levels were highest at the 1st decade and declined gradually with age. TC and TG levels of younger age (1st to 3rd decade) were equal to or even higher than those of Americans in 1972-76. It was concluded that serum lipid levels of Japanese have increased in the past 20 years and approached to the levels of Europeans and Americans.

```
response = qa_with_sources.invoke({
    "question": "Were there any studies done in Japan before 2000 regarding cholesterol levels? What year was it?"
})

```

Which results in the following output:

```
{
"answer": "A study was done in Japan in 1980 regarding cholesterol levels. "
"question": "Were there any studies done in Japan before 2000 regarding cholesterol levels? What year was it?"
"sources": 10.1253/jcj.47.1351"
}
```

### Step 5: Serve the Model

With our working pipeline, we're ready to serve our model.

Create an MLFlow Model
To prepare our model to be served with MLFlow, we'll create a class that contains two functions inside: (1) an initialization function and (2) an inference function.

MLFlow will initialize the model via load_context(), which will initialize our Pinecone vector store, our LLM, and a pipeline function to call the two of them. MLFlow will then call inference via predict(), as stated here.

```
class MyModelWrapper(mlflow.pyfunc.PythonModel):
    def load_context(self, context):
        # Initialize Pinecone Vector Index
        api_key = os.environ.get("PINECONE_API_KEY")
        vector_store = PineconeVectorStore(
            index=pc.Pinecone(api_key=api_key).Index(index_name),
            embedding=PineconeEmbeddings(model=model_name,
                pinecone_api_key=api_key
            )
        )

        # Completion LLM
        llm = ChatOpenAI(
            openai_api_key=os.environ['OPENAI_API_KEY'],
            model_name='gpt-3.5-turbo',
            temperature=0.0
        )

        # RAG & LLM Pipeline
        self.qa_with_sources = RetrievalQAWithSourcesChain.from_chain_type(
            llm=llm,
            chain_type="stuff",
            retriever=vector_store.as_retriever()
        )

    def predict(self, context, model_input):
        question = model_input[0]
        if not isinstance(question, str):
            question = str(question)
        response = self.qa_with_sources.invoke({
                "question": question
        })
        return response
```

Once we have defined our model class, export this model.

```
import shutil
from pathlib import Path

model_path = Path("my_model")

# Remove the older model directory if one exists.
if model_path.exists() and model_path.is_dir():
    shutil.rmtree(model_path)

my_model = MyModelWrapper()
mlflow.pyfunc.save_model(path=model_path, python_model=my_model)
```

Loading the MLFlow Model
As a quick test, let's load the model, then call inference with a sample question.

```
model = mlflow.pyfunc.load_model(model_path)
model.predict(["What is cholesterol?"])
```

which results in the following output

```
{'question': 'What is cholesterol?',
 'answer': 'Cholesterol is a 27-carbon steroid that is an essential component of the cell membrane and a precursor to steroid hormones. It is also involved in the formation of bile acids and very low-density lipoprotein. Cholesterol synthesis and absorption play a crucial role in the pathogenesis of dyslipidemias and cardiovascular diseases. \n',
 'sources': '10.1016/j.acmx.2015.12.002, 10.14797/mdcj-15-1-9, 10.1016/j.numecd.2011.04.010, 10.1016/0002-9149(88)90002-1'}
Serve the Model with MLFlow
Run this in your command line interface (CLI) to serve the model:
```

#### Serve the Model with MLFlow on your development machine

Run this in your command line interface (CLI) to serve the model:

`mlflow models serve -m my_model --port 8000 --no-conda --host 0.0.0.0`

Once the model has been successfully served, let's send a sample request.

```
import requests
requests.post("http://127.0.0.1:8000/invocations", json={'inputs': ["What is cholesterol?"]}).json()
```

output:

```
{'predictions': {'question': 'What is cholesterol?',
  'answer': 'Cholesterol is a 27-carbon steroid that is an essential component of the cell membrane and a precursor of steroid hormones. It is also involved in the formation of bile acids and very low-density lipoprotein. Cholesterol synthesis and absorption play a crucial role in the pathogenesis of dyslipidemias and cardiovascular diseases. \n',
  'sources': '10.1016/j.acmx.2015.12.002, 10.14797/mdcj-15-1-9, 10.1016/j.numecd.2011.04.010, 10.1016/0002-9149(88)90002-1'}}
```

### Deploy the MLflow pipelines

#### Copy the serialized model to network storage
To deploy the MLFlow pipeline, we first need to save the serialized model to a network location. To do so, you can copy the model to any network location. An [S3 bucket](/docs) or a [shared folder(managed NFS)](/docs). If you don't have access to either of these, you can use a layer Saturn Cloud provides on top of object storage called SaturnFS.

```
saturnfs cp --recursive my_model sfs://${SATURN_ORG}/${SATURN_USER}/my_model
```

#### Create a new deployment that serves the pipeline

1. Go back to the resources page and click on "New Deployment"
2. Under the command section, type `mlflow models serve -m my_model --port 8000 --no-conda --host 0.0.0.0`
3. Choose a GPU instance
4. Under the image, select the `saturn-python-llm` image, version `2024.08.01`
5. Click on "show advanced options" under the `Environment` section and add code to the start script to download your model
    ```
    saturnfs cp --recursive sfs://${SATURN_ORG}/${SATURN_USER}/my_model my_model
    ```
6. Click save
7. Click "start" to deploy your pipeline
