# Weights & Biases #
    mb = master_bar(range(n_epochs))
    for epoch in mb:
        count = 0
        model.train()

        for inputs, labels in progress_bar(train_loader, parent=mb):
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
            logs = {
                "train/train_loss": loss.item(),
                "train/learning_rate": scheduler.get_last_lr()[0],
                "train/correct": correct,
                "train/epoch": epoch + count / len(train_loader),
                "train/count": count,
            }

            # Occasionally some images to ensure the image data looks correct
            if count % 25 == 0:
                logs["examples/example_images"] = wandb.Image(inputs[:5], caption=f"Step: {count}")

            # Log some predictions to wandb during final epoch for analysis
            if epoch == max(range(n_epochs)) and count % 4 == 0:
                for i in range(len(labels)):
                    preds_table.add_data(wandb.Image(inputs[i]), labels[i], preds[i], perct[i])

            # Log metrics to wandb
            wandb.log(logs)

            count += 1

    # Upload your predictions table for analysis
    predictions_artifact = wandb.Artifact(
        "train_predictions_" + str(wandb.run.id), type="train_predictions"
    )
    predictions_artifact.add(preds_table, "train_predictions")
    wandb.run.log_artifact(predictions_artifact)

    # Close your wandb run
    wandb.run.finish()
```

### Run Model
You can now monitor the model run on Weights & Biases in real time.



```python
simple_train_single(**model_params)
```

At this point, you can view the Weights & Biases dashboard to see the performance of the model and system resources utilization in real time.

## Conclusion

This example showed how straightforward it is to use Weights & Biases while running Saturn Cloud. By only adding your API key to the credentials section of Saturn Cloud you can quickly use Weights & Biases in your resources.
