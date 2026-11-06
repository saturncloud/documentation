# Inference Quickstart

This page walks through deploying a model from the catalog and calling it. You need a Saturn Cloud account. If you don't have one, [sign up](/signup/).

## 1. Create the endpoint

1. In the Saturn Cloud app, open **Token Factory**, then **Endpoints**, and create a new endpoint.
2. Under **Serving tier**, choose **Dedicated GPU**.
3. Under **Serve**, choose **Base model (no fine-tuning)** and pick a model. A small model such as `Qwen/Qwen3-8B` loads quickly and fits on one GPU.
4. Leave **Serving topology** set to **Aggregated**.
5. Pick a GPU instance size. If the model has been profiled, the form warns you when the size has fewer GPUs than the model was profiled on.
6. Under **Details**, give the endpoint a name, then click **Create endpoint**.

Some models on Hugging Face are gated. For those, add a credential that exposes `HF_TOKEN`, from a Hugging Face account that has accepted the model's license, before you create the endpoint. The form tells you when a model needs one.

The platform then schedules GPUs and loads the weights. Large models take longer to load. When the endpoint's status shows **Running**, it is ready.

## 2. Try it in the Playground

Open the endpoint and select the **Playground** tab. It sends chat requests to the endpoint so you can check its output before writing any code.

## 3. Call it from code

You need a Saturn Cloud API token. Click your name in the sidebar, open **Access Keys**, and create a token. See [API Tokens](/docs) for the token types.

The endpoint's **Overview** tab shows its URL and an example request with the right model name filled in. The API is OpenAI-compatible, so the OpenAI Python SDK works as is:

```python
import os
from openai import OpenAI

client = OpenAI(
    base_url="https://<your-endpoint-url>/v1",
    api_key=os.environ["SATURN_TOKEN"],
)

response = client.chat.completions.create(
    model="<your-endpoint-name>",
    messages=[{"role": "user", "content": "Write a haiku about GPUs."}],
)
print(response.choices[0].message.content)
```

Or with `curl`:

```bash
curl https://<your-endpoint-url>/v1/chat/completions \
  -H "Authorization: Bearer $SATURN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"model": "<your-endpoint-name>", "messages": [{"role": "user", "content": "Hello"}]}'
```

## 4. Stop it when you're done

A dedicated endpoint holds its GPUs for as long as it runs, whether or not it gets traffic. Stop it from the endpoint list or the endpoint page when you no longer need it.

## Next steps

- [Calling an Endpoint](<docs/inference/calling-endpoints.md>) covers authentication, model names and streaming.
- [Inference Endpoints](<docs/inference/inference-endpoints.md>) covers disaggregated serving, autoscaling and serving fine-tuned adapters.
- For production traffic, [talk to us about dedicated GPU capacity](/request-a-demo/). We profile your model on the hardware it will run on and commit to the throughput and latency we measure.
