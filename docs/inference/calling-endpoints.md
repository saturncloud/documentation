# Calling an Endpoint|
| Dedicated endpoint serving a base model | The endpoint name |
| Dedicated endpoint serving a fine-tuned checkpoint | The endpoint name |
| Shared (serverless) endpoint | The name shown in the endpoint's example request |

The endpoint's **Overview** tab has an example request with the correct value filled in. `GET /v1/models` lists the names an endpoint accepts.

## Requests

### Chat completions

```python
import os
from openai import OpenAI

client = OpenAI(
    base_url="https://<your-endpoint-url>/v1",
    api_key=os.environ["SATURN_TOKEN"],
)

response = client.chat.completions.create(
    model="<your-endpoint-name>",
    messages=[
        {"role": "system", "content": "You answer in one sentence."},
        {"role": "user", "content": "What is tensor parallelism?"},
    ],
    max_tokens=200,
)
print(response.choices[0].message.content)
```

### Streaming

Set `stream=True` to receive tokens as they are generated:

```python
stream = client.chat.completions.create(
    model="<your-endpoint-name>",
    messages=[{"role": "user", "content": "Explain KV caching."}],
    stream=True,
)
for chunk in stream:
    print(chunk.choices[0].delta.content or "", end="", flush=True)
```

### curl

```bash
curl https://<your-endpoint-url>/v1/chat/completions \
  -H "Authorization: Bearer $SATURN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "<your-endpoint-name>",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

Request parameters such as `temperature`, `top_p`, `max_tokens` and `stop` work as in the OpenAI API. Features that depend on the model, such as tool calling and reasoning output, follow the model's chat template, so check the model card for what the model supports.

## Monitoring

The endpoint page has tabs for **Metrics** (request rate, latency, time to first token, token throughput, error rate and GPU use), **Logs** and **Usage**.
