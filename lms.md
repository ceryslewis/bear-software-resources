# TensorFlow Large Model Support Usage

## What is Large Model Support

TensorFlow Large Model Support (TFLMS) is a Python module that provides an approach to training large models and data that cannot normally be fitted into GPU memory. It takes a computational graph defined by users, and automatically adds swap-in and swap-out nodes for transferring tensors from GPUs to the host and vice versa. During training and inferencing this makes the graph execution operate like operating system memory paging. The system memory is effectively treated as a paging cache for the GPU memory and tensors are swapped back and forth between the GPU memory and CPU memory. 

Large Model Support is also available for PyTorch and Caffe, however these packages will not be explained here. 

## Memory issues with model training

There are three main reasons for which a model can lead to running out of memory:

 + Model depth/complexity = the number of layers and nodes in a neural network.
 + Data Size = Number of samples/features in the dataset used. 
 + Batch size = Number of samples that get propagated through a neural network.

One solution to this problem has traditionally been to reduce the model size by trying to get rid of less relevant features during the pre-processing stage. This can be done using either Feature Importance or Feature Extraction techniques (e.g. PCA, LDA). 
Using this approach can possibly lead to reduced noise (decreasing the chance of overfitting) and faster training times. One downside through to this approach can be a consistent decrease in accuracy. If the model needs a high complexity to capture all the important characteristics of a dataset, reducing the dataset size will in fact inevitably lead to worse performances. In this case, Large Model Support can be the solution to this problem.

## Sharing system memory

One of the typical short falls of GPU usage for compute applications is a lack of memory on the device. Whereas modern HPC systems typically have upwards of a 100GB of system memory, with some systems reaching over 1Tb, GPUs are still at the ~10GB level of on board memory. Data can be shared between system and device memory, however traditionally this transfer is handled by the PCIe bus which has both a low bandwidth and high latency compared to that of device memory itself. Therefore, GPU usage in compute has always tried to avoid transferring data between the device and system memory. However, it is at times necessary to make sure transfers, when models are too large to fit into device memory. Doing this is complicated and often requires extensive recoding of methods.

## Automatic configuration

With TFLMS the technical reorganisation and recoding of AI analyses has already been done. When using TFLMS, the memory required to perform the analyses is determined automatic by the TFLMS algorithms. This allow LMS to store the majority of AI data in the system memory transferring to the device what data is needed when it is needed. Moreover, these transfers take place asynchronously to the work. This means that as the device is performing calculations on one set of data, the previous data is being transferred out of memory (to system memory) and the next set of data is being transferred in, this saves the device from long wait times between compute cycles. 

## What are the advantages of LMS

TFLMS allows AI training and analyses to make use of GPU resources where they otherwise would be too large or too complex to do so considering the available memory on the GPU. On a typical system which connects GPU to CPU using a PCIe bus, TFLMS will allow the model to run, however, it will be slower than a hypothetical system where the same size model could (somehow) be held entirely in device memory. This is because even though the device-system transfers are asynchronous, the PCIe transfer speed is sufficiently slow to mean that the GPU will still have to wait. How long the GPU has to wait will be based on the PCIe generation, size of the model, number of CUDA cores on the GPU to name a few. 

## Advantage of TFLMS with POWER9

The IBM POWER9 systems available at the University of Birmingham solves one of these above limitations, that being the transfer speed of the PCIe bus. POWER9 systems do not connect the GPU to the system through the PCIe bus, but instead though a new interconnect known as NVLINK. NVLINK was developed by Nvidia to facilitate low latency, high bandwidth transfers between multiple GPUs as well as between GPUs and system memory. NVLINK is effectively 10 times faster than PCIe 3.0. It real world terms this means that the time spent by the GPU waiting for the asynchronous transfer of data on and off the GPU is lower, which makes more efficient use of the GPU compute power.

## Examples

Below are 4 examples of TFLMS which are intended to demostrate the benefit and usage of TFLMS as well as to be instructive in helping users to modifying their existing Tensorflow code to take advantage of TFLMS. All exmaples can be found in the bbexamples directory on BlueBEAR, information on accessing this directory can be found [here](https://intranet.birmingham.ac.uk/it/teams/infrastructure/research/bear/bluebear/bluebear-job-submission.aspx) 

### Example 1 - Adjustable image Resolution

This example uses the ancillary library for Tensorflow, Keras. This example generates sample data to train a model. This data is too large to fit in onboard memory of a V100 Nvidia GPU which has approximately 16GB. Therefore standard model training will fail with an out of memory error. However the addition of --lms to the command line call TFLMS will detect the limited memory, self determine the required memory transfers and proceed with the training, exiting successfully.

### Example 2 - Session based training example

The mnist_deep_lms.py file is an example of how to enable TFLMS when using Session based training. This example is a TFLMS enabled version of this TensorFlow example.

### Example 3 - Estimator based training example

The cnn_mnist_lms.py file is an example of how to enable TFLMS when using Estimator based training. This example is a TFLMS enabled version of this TensorFlow example.

### Example 4 - Tensorflow Keras based training example

The mnist_cnn_keras.py file is an example of how to enable TFLMS when using TensorFlow Keras based training. This example is a TFLMS enabled version of this Keras example.
