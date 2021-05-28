# Train a Model with PyTorch - CNN for Image Classification #   
        with torch.no_grad():
            model.eval()  # Set model to evaluation mode
            for inputs_t, labels_t in dataloaders["val"]:
                t_count += 1

                # Pass items to GPU
                inputs_t = inputs_t.to(device)
                labels_t = labels_t.to(device)

                # Run model iteration
                outputs_t = model(inputs_t)

                # Format results
                _, pred_t = torch.max(outputs_t, 1)
                perct_t = [torch.nn.functional.softmax(el, dim=0)[i].item() for i, el in zip(pred_t, outputs_t)]

                loss_t = criterion(outputs_t, labels_t)
                correct_t = (pred_t == labels_t).sum().item()

        scheduler.step(loss)
```

### Assign Parameters

```python
num_workers = 64
client.restart() # Optional, but recommended - clears cluster memory

startparams = {'n_epochs': 100, 
                'batch_size': 100,
                'train_pct': .8,
                'base_lr': 0.01,
                'imagenetclasses':imagenetclasses,
                'subset': False,
                'n_workers': 1}
```

### Run training workflow on cluster

Now we've done all the hard work, and just need to run our function! Using `dispatch.run` from `dask-pytorch-ddp`, we pass in the learning function so that it gets distributed correctly across our cluster. This creates futures and starts computing them.

Inside the `dispatch.run()` function in `dask-pytorch-ddp`, we are actually using the `client.submit()` method to pass tasks to our workers, and collecting these as futures in a list. We can prove this by looking at the results, here named "futures", where we can see they are in fact all pending futures, one for each of the workers in our cluster.

> Why don't we use `.map()` in this function?  
> Recall that `.map` allows the Cluster to decide where the tasks are completed - it has the ability to choose which worker is assigned any task. That means that we don't have the control we need to ensure that we have one and only one job per GPU. This could be a problem for our methodology because of the use of DDP.
> Instead we use `.submit` and manually assign it to the workers by number. This way, each worker is attacking the same problem - our training problem - and pursuing a solution simultaneously. We'll have one and only one job per worker.

```python
futures = dispatch.run(
    client, 
    run_training, 
    bucket = "saturn-public-data", 
    prefix = "dogs/Images", 
    **startparams
)
futures
```

You may want to add steps in the workflow that checkpoint the model and/or save performance statistics. There are several ways to do this, including saving those metrics to the workers for later retrieval, or writing them to an external data store such as S3. If you need help with this, contact our support!