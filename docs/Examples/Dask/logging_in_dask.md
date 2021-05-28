# Logging in Dask
distributed.core - INFO - Starting established connection
distributed.worker - INFO - Starting Worker plugin saturn_setup
distributed.worker - INFO - Computed exponent 11^12 = 3138428376721
distributed.worker - INFO - Computed exponent 5^6 = 15625
```

There we correctly see that the logs included the info logging we did within the function. That concludes the example of how to generate logs from within Dask. This can be a great tool for understanding how code is running, debugging code, and better propagating warnings and errors.
