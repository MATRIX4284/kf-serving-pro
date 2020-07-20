# KFServing
KFServing provides a Kubernetes [Custom Resource Definition](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) for serving machine learning (ML) models on arbitrary frameworks. It aims to solve production model serving use cases by providing performant, high abstraction interfaces for common ML frameworks like Tensorflow, XGBoost, ScikitLearn, PyTorch, and ONNX.

It encapsulates the complexity of autoscaling, networking, health checking, and server configuration to bring cutting edge serving features like GPU Autoscaling, Scale to Zero, and Canary Rollouts to your ML deployments. It enables a simple, pluggable, and complete story for Production ML Serving including prediction, pre-processing, post-processing and explainability.

![KFServing](/docs/diagrams/kfserving.png)

We will use the latest stable tag=0.2.2 as of now in this tutorial: git clone https://github.com/kubeflow/kfserving-pro
Go to Folder containing the Docker needed to build pytorchserver with your custom model.
system@AI-NODE1:~/kf-serving-pro$ cd python
Prepare the Custom CPU Docker image:
system@AI-NODE1:~/kf-serving-pro/python$ docker build -t kaustav4284/pytorchserver:latest -f pytorch.Dockerfile .Sending build context to Docker daemon 1.568MBStep 1/14 : FROM ubuntu:18.04 ---> d27b9ffc5667Step 2/14 : ARG PYTORCH_VERSION=1.3.1 ---> Using cache -- -> 01bf51c4ae73
cd kfserving
cd install/0.2.2 nano kfserving.yaml
“pytorch”: { “image”: “gcr.io/kfserving/pytorchserver”, “defaultImageVersion”: “0.2.2”, “allowedImageVersions”: [ “0.2.2” ] }


