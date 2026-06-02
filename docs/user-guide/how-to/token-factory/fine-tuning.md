# Fine-Tuning Jobs|
| `pending` | Scheduled, waiting for a GPU node. |
| `running` | Training. |
| `stopping` | Cancellation in progress. |
| `stopped` | Cancelled before completion. |
| `completed` | Finished successfully. A checkpoint is available. |
| `error` | Failed. No usable checkpoint. |

## What the job view shows

The job view is an explicit, curated set of fields. It shows the job's name, status,
base model, dataset, the hyperparameters you submitted, and a `checkpoint` reference once
training completes. The hyperparameters are read back from the rendered training configuration,
so the numbers on the view always match the numbers you submitted (including
`effective_batch_size`, reconstructed from the realized gradient-accumulation and micro-batch
values).

<img src="/images/docs/token-factory/fine-tuning-job-overview.png" alt="Fine-tuning job detail page showing configuration, hyperparameters, timing, and an experiment-tracking link" class="doc-image">

The job's Metrics tab plots training and evaluation loss, learning rate, throughput, and GPU
utilization as the run progresses.

<img src="/images/docs/token-factory/fine-tuning-job-metrics.png" alt="Fine-tuning job metrics tab showing training and eval loss, learning rate, gradient norm, throughput, and GPU charts" class="doc-image">

The view deliberately does not expose the underlying deployment's environment variables,
command, or image. Because you can place secrets in a job's environment, the view never echoes
those back.

## The checkpoint

A successful job produces at most one checkpoint, registered automatically when training
finishes. The job view's `checkpoint` field points at it once it is ready; until then the field
is empty. The checkpoint records which job produced it, giving you lineage from a served model
back to the exact run and hyperparameters.

If a job fails or is killed before it finishes (out-of-memory, eviction, node loss), no
checkpoint is registered as ready, and the job view reflects the failure. The platform
reconciles the job's final state regardless of how it ended.

## Experiment tracking

If your organization has an experiment tracker configured (Weights & Biases, MLflow, or Comet), runs are
logged to it automatically and tagged with the job's identity, so you can deep-link from a job
to its tracker run. If no tracker is configured, nothing is logged and the job runs normally.
Experiment tracking is optional and has no effect on whether a job succeeds.

## Next step

Once a job has produced a checkpoint, serve it as an [Inference Endpoint](<docs/user-guide/how-to/token-factory/inference-endpoints.md>).
