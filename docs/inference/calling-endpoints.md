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

### Video input

Models that take video, such as the Qwen3-VL models in the catalog, accept a `video_url` content part in a chat completions request. The model card in the catalog lists **video** under its inputs. Video models run on dedicated endpoints only.

```python
response = client.chat.completions.create(
    model="<your-endpoint-name>",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "video_url", "video_url": {"url": "https://example.com/clip.mp4"}},
                {"type": "text", "text": "What happens in this video?"},
            ],
        }
    ],
    max_tokens=300,
)
```

The `video_url` part is an extension that vLLM defines; OpenAI's own chat API has no video type. The OpenAI Python SDK sends the part as written.

- **Send an https URL.** The endpoint downloads the video itself, so the URL must use https and be reachable from the public internet without a login. Plain `http://` URLs and addresses inside the cluster or a private network are refused.
- **Base64 only for short clips.** A `data:video/mp4;base64,...` URL also works, but the whole request body is limited to 64 MiB, and base64 adds a third to the size. A video larger than about 45 MiB does not fit.
- **One video per request** by default. Each turn of a conversation re-sends the earlier messages, so a conversation can hold one video. To allow more, set the `limit-mm-per-prompt.video` engine option when you create the endpoint; each video adds work for the model.
- **Billing is in tokens.** The video's frames become input tokens and are counted in `usage.prompt_tokens`. A 10-second clip can be several thousand tokens.

The endpoint's **Playground** tab has a **Video URL** field for video models.

Request parameters such as `temperature`, `top_p`, `max_tokens` and `stop` work as in the OpenAI API. Features that depend on the model, such as tool calling and reasoning output, follow the model's chat template, so check the model card for what the model supports.

## Monitoring

The endpoint page has tabs for **Metrics** (request rate, latency, time to first token, token throughput, error rate and GPU use), **Logs** and **Usage**.
