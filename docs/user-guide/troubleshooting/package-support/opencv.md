# OpenCV (Python)

The process below will allow you to use the computer vision library [OpenCV](https://opencv.org/) on Saturn Cloud

Create a new Jupyter server resource and when doing so set the following options: 

* Set it to use a **GPU**. For the image, in theory any should work but this has been validated with **saturn-pytorch** so we recommend that.
* Add the commands below to the startup script. This will install OpenCV when starting your resource:

```console
sudo apt-get update
sudo apt-get install -y libgl1-mesa-glx
conda install mamba -n base -y -c conda-forge
mamba install opencv=4.5.3 -y -c conda-forge
```

The startup script installs a necessary linux library for using opencv, then installs opencv itself. The script uses mamba instead of conda to install opencv since it is faster. You could instead install opencv from conda but it would take longer for the resource to start up.

Once the resource has started, you can verify that the installation succeeded by running:

```python
import cv2
```