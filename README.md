# kubernetes-taw

This tutorial provides a walkthrough of Kubernetes fundamentals.

## Costs

This tutorial uses billable components of Google Cloud Platform, including:

* Google Kubernetes Engine
* Google Cloud Load Balancing

Use the [Pricing Calculator][pricing] to generate a cost estimate based on your
projected usage.

[pricing]: https://cloud.google.com/products/calculator

## Before you begin

1. Create a [Google Cloud Platform project](https://console.cloud.google.com/project).

2. Enable [billing](https://console.cloud.google.com/billing) for the project.

3. Enable the [Google Kubernetes Engine API](https://console.cloud.google.com/flows/enableapi?apiid=container.googleapis.com).

4. Open [Cloud Shell](https://console.cloud.google.com/cloudshell).

## Setup

1. Clone and cd to this repo:

```
git clone https://github.com/theckang/kubernetes-taw
cd kubernetes-taw
```

2. Create the Kubernetes Engine cluster:

```
gcloud container clusters create cluster-1 --zone us-west1-a
```

3. Set the zone of `cluster-1`:

```
gcloud config set compute/zone us-west1-a
```

4. Get authentication credentials to `cluster-1`:

```
gcloud container clusters get-credentials cluster-1
```

## Pod

### Basic Example

Review the YAML:

    cat pod.yaml
    cat busybox.yaml

Deploy with Kubernetes:

    kubectl create -f pod.yaml
    kubectl create -f busybox.yaml

Get pod:

    kubectl get pod pod-example

Test webserver:

    POD_IP=$(kubectl get pod pod-example -o go-template='{{.status.podIP}}')
    kubectl exec busybox curl $POD_IP
    
### Sidecar Example

Review the YAML:

    cat sidecar.yaml

Deploy with Kubernetes:

    kubectl create -f sidecar.yaml

Get pod:

    kubectl get pod sidecar-example

Test webserver:

    SIDECAR_IP=$(kubectl get pod sidecar-example -o go-template='{{.status.podIP}}')
    kubectl exec busybox curl $SIDECAR_IP
    
Check if sidecar has access to logs:

    kubectl exec sidecar-example --container logrotate ls /logs
    kubectl exec sidecar-example --container logrotate cat /logs/access.log

### Init Example

Review the YAML:

    cat init.yaml

Deploy with Kubernetes:

    kubectl create -f init.yaml

Get pod:

    kubectl get pod init-example

Check if web server content was initialized:

    INIT_IP=$(kubectl get pod init-example -o go-template='{{.status.podIP}}')
    kubectl exec busybox curl $INIT_IP

Check if sidecar has access to logs:

    kubectl exec sidecar-example --container logrotate ls /logs
    kubectl exec init-example --container logrotate cat /logs/access.log

## Deployment

Delete the basic pod deployed earlier and observe the behavior:

    kubectl delete pod pod-example

Review the YAML:

    cat deployment.yaml

Deploy with Kubernetes:

    kubectl create -f deployment.yaml

List deployment:

    kubectl get deploy nginx-deployment

List pods in the deployment:

    kubectl get pods -l app: nginx

Delete one of the pods in the deployment:
    
    POD_ID=$(kubectl get pods -l "app=nginx")
    kubectl get delete pod POD_IP

Watch Kubernetes heal the deployment:

    watch kubectl get pods -l "app=nginx"

Scale the deployment:

    kubectl scale --replicas=3 deployment/nginx-deployment

Get pods in the deployment:

    kubectl get pods -l "app=nginx"

## Services

Review the YAML:

    cat svc.yaml

Deploy the service:

    kubectl create -f svc.yaml

You can also expose the service with:

    # kubectl expose deploy nginx-deployment --port=80 --target-port=80

Watch service until EXTERNAL_IP is set:

    watch kubectl get svc nginx-svc

Get endpoint:

    kubectl get endpoints

Test service:

    EXTERNAL_IP=kubectl get svc -l "app=nginx-svc" # update
    curl EXTERNAL_IP

Scale deployment down:

    kubectl scale --replicas=1 deployment/nginx-deployment

Watch Kubernetes dynamically update endpoint:

    watch kubectl get endpoints
