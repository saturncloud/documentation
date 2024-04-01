# Dispatching Jobs for Research Workflows
id-hugo-hello-world-a387e542a27a4e689f16a0fac48901de-nz-0-hplcq    pending      live          2024-04-01T15:29:36+00:00
id-hugo-hello-world-a387e542a27a4e689f16a0fac48901de-wc-0-4ptfv    completed    historical    2024-04-01T01:29:46+00:00    2024-04-01T01:32:31+00:00
id-hugo-hello-world-a387e542a27a4e689f16a0fac48901de-hd-0-w5qx4    completed    historical    2024-04-01T01:15:48+00:00    2024-04-01T01:18:34+00:00

```

You can then request the logs for each pod. Note, Saturn Cloud captures live and historical logs. Live logs are stored on the machine where the job is running. These disappear when the machine is torn down. Historical logs are an archive of the live logs, but there may be a few minute delay before logs end up in the historical log store. As a result, the CLI lets you specify which source you would like to choose for logs. If you omit the source, the client attempts to figure out the best source.

```
$ sc logs job hello-world id-hugo-hello-world-a387e542a27a4e689f16a0fac48901de-nz-0-hplcq
$ sc logs job hello-world id-hugo-hello-world-a387e542a27a4e689f16a0fac48901de-nz-0-hplcq --source live
$ sc logs job hello-world id-hugo-hello-world-a387e542a27a4e689f16a0fac48901de-nz-0-hplcq --source historical
```
