We will use the latest stable tag=0.2.2 as of now in this tutorial: git clone https://github.com/kubeflow/kfserving-pro
Go to Folder containing the Docker needed to build pytorchserver with your custom model.
system@AI-NODE1:~/kf-serving-pro$ cd python
Prepare the Custom CPU Docker image:
system@AI-NODE1:~/kf-serving-pro/python$ docker build -t kaustav4284/pytorchserver:latest -f pytorch.Dockerfile .Sending build context to Docker daemon 1.568MBStep 1/14 : FROM ubuntu:18.04 ---> d27b9ffc5667Step 2/14 : ARG PYTORCH_VERSION=1.3.1 ---> Using cache -- -> 01bf51c4ae73
cd kfserving
cd install/0.2.2 nano kfserving.yaml
“pytorch”: { “image”: “gcr.io/kfserving/pytorchserver”, “defaultImageVersion”: “0.2.2”, “allowedImageVersions”: [ “0.2.2” ] }
