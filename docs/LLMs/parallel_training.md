# Multi-Node Multi-GPU Parallel Training

### **Example Code**
```python
import tensorflow as tf

# Define MultiWorkerMirroredStrategy
strategy = tf.distribute.MultiWorkerMirroredStrategy()

# Define model and dataset within the strategy scope
with strategy.scope():
    model = tf.keras.Sequential([...])
    model.compile(optimizer='adam', loss='sparse_categorical_crossentropy')

dataset = tf.data.Dataset.from_tensor_slices(...).batch(32)

# Train the model
model.fit(dataset, epochs=10)
```

By leveraging `tf.distribute`, TensorFlow makes distributed training accessible, powerful, and highly customizable for multi-node, multi-GPU workloads.
