# Train a Neural Network Using Flux


## Overview

This example describes how to run a neural learning workflow using the famous MNIST dataset of handwritten digits. Flux.jl is a powerful Julia library for many types of machine learning, including neural networks. We will training a neural network on images of handwritten numbers to create an image classifier. 

## Modeling Process

### Imports
This exercise uses [Flux](https://fluxml.ai/Flux.jl/stable/) to train a simple neural network. The code loads several functions from Flux as well as some base functions.


```julia
using Flux, Flux.Data.MNIST
using CUDA
using Flux: onehotbatch, argmax, crossentropy, throttle, @epochs
using Base.Iterators: repeated
using Images
```

We can then check to see if CUDA is enabled. This will allow the model to run on the GPU instead of the CPU. This is potentially faster to train, but the speedup depends on a lot of factors.


```julia
CUDA.functional()
```

### Download and Examine the Dataset
First, we need to download the dataset. Flux has several built-in datasets, including MNIST. We can then take a look at the first image to see the type of data we are dealing with.


```julia
imgs = MNIST.images();
colorview(Gray, imgs[1])
```


```julia
labels = MNIST.labels();
labels[1]
```

We next convert the data into float32s to speed up training and reduce memory footprint while retaining precision.


```julia
float32converter(X) = Float32.(X)
f32_imgs = float32converter.(imgs)
```

We then convert the individual images to a single vector of size 784 (28x28) by 60000 (the number of images).


```julia
vectorize(x) = x[:]
vectorized_imgs = vectorize.(f32_imgs);
```


```julia
X = hcat(vectorized_imgs...)
size(X)
```

To look at only one image, we have to select a column and reshape.


```julia
one_image = X[:,1]
image_1 = reshape(one_image,28,28)
colorview(Gray,image_1)
```

Lastly, we transform the labels from digits to one-hot vectors. For example, if the label is 3, the y value will be [0 0 0 1 0 0 0 0 0 0].


```julia
y = onehotbatch(labels, 0:9)
```

### Build and Train the Model
Now we will actually build out neural network model. We will use a 32 node hidden layer and a 10 node output layer.


```julia
model = Chain(
        Dense(28^2, 32, relu),
        Dense(32, 10),
        softmax)
```

If we take a look at the output of the model now, we can see that it was initialized without any knowledge of the input. Each digit is approximately equally likely as an output.


```julia
model(one_image)
```

We then move the data to the GPU for processing. This is accomplished using the `fmap` function for the model and the `cu` function for the data.


```julia
X = cu(X)
y = cu(y)
model = fmap(cu, model)
```

Next, we will set up our loss and optimization functions. We also create an function to display the loss at each step and define the parameters of the model.


```julia
loss(x, y) = Flux.crossentropy(model(x), y)
opt = ADAM()
evalcb = () -> @show(loss(X, y))
ps = Flux.params(model);
```

We will then repeat the data we send to the neural network. This is a simple way to give the network more chances for corrections.


```julia
dataset_x = repeated((X, y), 200)
C = collect(dataset_x);
```

And finally we train the model for 10 epochs on the GPU. You can type `watch nvidia-smi` in a terminal to see the GPU utilization.


```julia
@epochs 10 Flux.train!(loss, ps, dataset_x, opt, cb = throttle(evalcb, 10))
```

We can check how the model performs on the test set. We move the model back to the CPU for this step. 

Here we simply choose the first image to check. The maximum value of the model output is at the index corresponding to the digit 7 (which aligns with the actual image).


```julia
X_test = hcat(float.(reshape.(MNIST.images(:test), :))...);
model = model |> cpu

test_image = model(X_test[:,1])
float32converter.(test_image)
```


```julia
argmax(test_image) - 1
```


```julia
test_image_1 = reshape(X_test[:,1],28,28)
colorview(Gray, test_image_1)
```

## Conclusion
Using Julia and the Flux package makes creating and training a simple neural network easy. Check out the rest of the [Flux documentation](https://fluxml.ai/Flux.jl/stable/) to see how to extend this process to more complex examples.
