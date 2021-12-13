# Weights & Biases (with Dask Cluster) #
    for epoch in range(n_epochs):
        count = 0
        model.train()

        for inputs, labels in train_loader:
            # zero the parameter gradients
            optimizer.zero_grad()

            inputs, labels = inputs.to(device), labels.to(device)

            # Run model iteration
            outputs = model(inputs)

            # Format results
            pred_idx, preds = torch.max(outputs, 1)
            perct = [
                torch.nn.functional.softmax(el, dim=0)[i].item() for i, el in zip(preds, outputs)
            ]

            loss = criterion(outputs, labels)
            correct = (preds == labels).sum().item()

            loss.backward()
            optimizer.step()
            scheduler.step()

            # Log your metrics to wandb
            if worker_rank == 0:
                logs = {
                    "train/train_loss": loss.item(),
                    "train/learning_rate": scheduler.get_last_lr()[0],
                    "train/correct": correct,
                    "train/epoch": epoch + count / len(train_loader),
                    "train/count": count,
                }

                # Occasionally some images to ensure the image data looks correct
                if count % 25 == 0:
                    logs["examples/example_images"] = wandb.Image(
                        inputs[:5], caption=f"Step: {count}"
                    )

                # Log some predictions to wandb during final epoch for analysis
                if epoch == max(range(n_epochs)) and count % 4 == 0:
                    for i in range(len(labels)):
                        preds_table.add_data(wandb.Image(inputs[i]), labels[i], preds[i], perct[i])

                # Log metrics to wandb
                wandb.log(logs)

            count += 1

    # Upload your predictions table for analysis
    if worker_rank == 0:
        predictions_artifact = wandb.Artifact(
            "train_predictions_" + str(wandb.run.id), type="train_predictions"
        )
        predictions_artifact.add(preds_table, "train_predictions")
        wandb.run.log_artifact(predictions_artifact)

        # Close your wandb run
        wandb.run.finish()
```

### Run Model

To run the model, we use the `dask-pytorch-ddp` function `dispatch.run()`. This takes our client, our training function, and our dictionary of model parameters. You can monitor the model run on all workers using the Dask dashboard, or monitor the performance of Worker 0 on Weights & Biases.


```python
client.restart()  # Clears memory on cluster- optional but recommended.
```


```python
%%time
futures = dispatch.run(client, simple_train_cluster, **model_params)
futures
```


```python
# If one or more worker jobs errors, this will describe the issue
futures[0].result()
```

At this point, you can view the Weights & Biases dashboard to see the performance of the model and system resources utilization in real time!

## Conclusion

From this example we were able to see that by using Weights & Biases you can monitor performance of each work in a Dask cluster on Saturn Cloud. Adding Weights & Biases to a Dask cluster is just as easy as adding it to a single machine, so this can be a great tool for monitor models
