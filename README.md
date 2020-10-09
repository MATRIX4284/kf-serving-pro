Kubeflow comes with a line of lineup of model serving.
We are goung to discuss about the kfserving module and how it can be utilized to serve a Pytorch model which can be auto-scaled,or zero scaled with the resource utilizaion changing dynamics.
THE BIGGEST CHALLENGE!!!
The biggest challenge is the Kubeflow Docs which does not provide a clear picture on how one can run a custom model (not the pytorch-cifar-10 given in the example).To run a custom model it is necessary to use a custom image which can configured according to application requirement.Kubeflow dcumentation really lacks the clarity making the execution of custom images on Kubeflow KFSERVING Pytorch Server almost REACH A DEAD END …OF COURSE IF YOU ARE NOT USING GOOGLE CLOUD.
Now I will give a step by step instruction how to run a Kubeflow Kfserving Pytorch Mode Service with a custom image.
We will use the latest stable tag=0.2.2 as of now in this tutorial:
git clone https://github.com/kubeflow/kfserving-pro
Go to Folder containing the Docker needed to build pytorchserver with your custom model.
system@AI-NODE1:~/kf-serving-pro$ cd python
Prepare the Custom CPU Docker image:
system@AI-NODE1:~/kf-serving-pro/python$ docker build -t kaustav4284/pytorchserver:latest -f pytorch.Dockerfile .
Sending build context to Docker daemon  1.568MB
Step 1/14 : FROM ubuntu:18.04
 ---> d27b9ffc5667
Step 2/14 : ARG PYTORCH_VERSION=1.3.1
 ---> Using cache
 ---> 01bf51c4ae73
cd kfserving
cd install/0.2.2
nano kfserving.yaml
“pytorch”: { “image”: “gcr.io/kfserving/pytorchserver”, “defaultImageVersion”: “0.2.2”, “allowedImageVersions”: [ “0.2.2” ] }
Replace the default docker image name from gcr.io/kfserving/pytorchserver to your custom docker image name e.g. abcd/pytorchServer and replace the “defaultImageVersion”: & “allowedImageVersions” with the necessary label .
Save the file.
Execute the file:
export TAG=0.2.2
Delete Previous Instance of kfserving:
export TAG=0.2.2
system@AI-NODE1:~/kf-serving-pro$ kubectl delete -f ./install/$TAG/kfserving.yaml
namespace "kfserving-system" deleted
customresourcedefinition.apiextensions.k8s.io "inferenceservices.serving.kubeflow.org" deleted
clusterrole.rbac.authorization.k8s.io "kfserving-proxy-role" deleted
clusterrole.rbac.authorization.k8s.io "manager-role" deleted
clusterrolebinding.rbac.authorization.k8s.io "kfserving-proxy-rolebinding" deleted
clusterrolebinding.rbac.authorization.k8s.io "manager-rolebinding" deleted
configmap "inferenceservice-config" deleted
Start the kfserving service:
system@AI-NODE1:~/kf-serving-pro$ kubectl create -f ./install/$TAG/kfserving.yaml
namespace/kfserving-system created
customresourcedefinition.apiextensions.k8s.io/inferenceservices.serving.kubeflow.org created
clusterrole.rbac.authorization.k8s.io/kfserving-proxy-role created
clusterrole.rbac.authorization.k8s.io/manager-role created
clusterrolebinding.rbac.authorization.k8s.io/kfserving-proxy-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/manager-rolebinding created
configmap/inferenceservice-config created
secret/kfserving-webhook-server-secret created
service/kfserving-controller-manager-metrics-service created
service/kfserving-controller-manager-service created
statefulset.apps/kfserving-controller-manager created
Check if your custom image is reflecting:
kubectl edit cm inferenceservice-config -n kfserving-system
predictors: |-
    {
        "tensorflow": {
            "image": "tensorflow/serving",
            "defaultImageVersion": "1.14.0",
            "defaultGpuImageVersion": "1.14.0-gpu",
            "allowedImageVersions": [
               "1.11.0",
               "1.11.0-gpu",
               "1.12.0",
               "1.12.0-gpu",
               "1.13.0",
               "1.13.0-gpu",
               "1.14.0",
               "1.14.0-gpu"
            ]
        },
        "onnx": {
            "image": "mcr.microsoft.com/onnxruntime/server",
            "defaultImageVersion": "v0.5.1",
            "allowedImageVersions": [
               "v0.5.1"
            ]
        },
        "sklearn": {
            "image": "gcr.io/kfserving/sklearnserver",
            "defaultImageVersion": "0.2.2",
            "allowedImageVersions": [
               "0.2.2"
            ]
        },
        "xgboost": {
            "image": "gcr.io/kfserving/xgbserver",
            "defaultImageVersion": "0.2.2",
            "allowedImageVersions": [
               "0.2.2"
            ]
        },
        "pytorch": {
            "image": "kaustav4284/pytorchserver",
            "defaultImageVersion": "latest",
            "allowedImageVersions": [
               "latest"
            ]
If the Custom image is reflecting .
Go to the folder having the application manifest:
system@AI-NODE1:~/kf-serving-pro$ cd docs/samples/pytorch
Run the yaml of the application:
system@AI-NODE1:~/kf-serving-pro/docs/samples/pytorch$ kubectl create -f pytorch.yaml
inferenceservice.serving.kubeflow.org/pytorch-cifar10-folder created
Check if the execution is successful:
system@AI-NODE1:~/test/kfserving/docs/samples/pytorch$ kubectl get po -n default
NAME                                                              READY   STATUS      RESTARTS   AGE
pytorch-cifar10-predictor-default-8t8gd-deployment-8948964g27bg   2/2     Running     0          10s
pytorch-dist-mnist-gloo-master-0                                  0/1     Completed   0          15d
pytorch-dist-mnist-gloo-worker-0                                  0/1     Completed   0          15d
Check the Log of the running container hosting the Pytorch Model:
system@AI-NODE1:~/test/kfserving/docs/samples/pytorch$ kubectl logs pytorch-cifar10-predictor-default-8t8gd-deployment-8948964g27bg -c kfserving-container
[I 200709 01:43:36 storage:35] Copying contents of /mnt/models to local
[I 200709 01:43:36 kfserver:88] Registering model: pytorch-cifar10
[I 200709 01:43:36 kfserver:77] Listening on port 8080
[I 200709 01:43:36 kfserver:79] Will fork 0 workers
[I 200709 01:43:36 process:126] Starting 6 processes
Get the service names for pytorch-cifar10-folder model:
system@AI-NODE1:~/kf-serving-pro/docs/samples/pytorch$ kubectl get svc
NAME                                                     TYPE           CLUSTER-IP     EXTERNAL-IP                                            PORT(S)                             AGE
kubernetes                                               ClusterIP      10.0.0.1       <none>                                                 443/TCP                             28d
pytorch-cifar10-folder-predictor-default                 ExternalName   <none>         cluster-local-gateway.istio-system.svc.cluster.local   <none>                              3m
pytorch-cifar10-folder-predictor-default-9kh5q           ClusterIP      10.0.7.202     <none>                                                 80/TCP                              3m8s
pytorch-cifar10-folder-predictor-default-9kh5q-private   ClusterIP      10.0.240.247   <none>                                                 80/TCP,9090/TCP,9091/TCP,8022/TCP   3m8s
pytorch-dist-mnist-gloo-master-0                         ClusterIP      None           <none>                                                 23456/TCP                           24d
Port Forward the Pytorch Model Server for Prediction:
system@AI-NODE1:~/kf-serving-pro/docs/samples/pytorch$ kubectl port-forward -n default svc/pytorch-cifar10-folder-predictor-default-sdq2l-private 15019:80
Forwarding from 127.0.0.1:15019 -> 8012
Forwarding from [::1]:15019 -> 8012
Handling connection for 15019
Handling connection for 15019
Handling connection for 15019
Run the Prediction with the new model pytorch-cifar10-folder:
system@AI-NODE1:~/kf-serving-pro/docs/samples/pytorch$ curl -v -H "Host: localhost" -d @./input.json http://localhost:15019/v1/models/pytorch-cifar10-folder:predict
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 15019 (#0)
> POST /v1/models/pytorch-cifar10-folder:predict HTTP/1.1
> Host: localhost
> User-Agent: curl/7.47.0
> Accept: */*
> Content-Length: 110681
> Content-Type: application/x-www-form-urlencoded
> Expect: 100-continue
> 
< HTTP/1.1 100 Continue
* We are completely uploaded and fine
< HTTP/1.1 200 OK
< Content-Length: 223
< Content-Type: application/json; charset=UTF-8
< Date: Sat, 18 Jul 2020 02:57:58 GMT
< Server: TornadoServer/6.0.4
< 
* Connection #0 to host localhost left intact
{"predictions": [[-1.3545894622802734, -2.094174385070801, 0.3954501748085022, 3.433469295501709, -0.08178478479385376, 1.7413527965545654, 0.9827516674995422, -1.3434531688690186, 0.63302546





