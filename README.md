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

1. Enable [billing](https://console.cloud.google.com/billing) for the project.

1. Enable the [Google Kubernetes Engine API](https://console.cloud.google.com/flows/enableapi?apiid=container.googleapis.com).

1. Open [Cloud Shell](https://console.cloud.google.com/cloudshell).

## Setup

1. Create the Kubernetes Engnie cluster:

    gcloud container clusters create cluster-1 --zone us-west1-a

1. Set the zone of `cluster-1`:

    gcloud config set compute/zone us-west1-a

1. Get authentication credentials to `cluster-1`:

    gcloud container clusters get-credentials cluster-1

## Pod

Deploy a nginx web application container in a pod.  Create the YAML for the pod:

    cat > pod.yaml <<EOF
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-pod
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.12.2
        ports:
        - containerPort: 80
    EOF

Deploy the pod in Kubernetes:

    kubectl apply -f pod.yaml


Outline:
* What just happened?  Expand.
* That was easy.  What about the git synchronizer?  git synchronizer: https://github.com/kubernetes/git-sync
* Would another pod work?  Deploy it.  No, it will not work because you need to share logs on the same filesystem.
* One pod with two containers!
* Deploy again.
* This is the sidecar pattern.  1:1 between app and synchronizer.  You can reuse the synchronizer with other containers like nodejs, etc.

## Deployment

    cat > apiVersion: apps/v1 <<EOF
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      selector:
        matchLabels:
          app: nginx
      replicas: 1
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.12.2
            ports:
            - containerPort: 80
    EOF


Outline
* What happens if my pod dies?  It goes away.  But I really want Kubernetes to heal.
* Use a deployment!  See the replica set, replication controller under the hood.
* What if I want multiple copies of the pod?  Scale instantly!  x3

## Services

Let's take a step back with our docker containers.  How would I expose 3 containers to the outside world?


Outline
* Show manually how to with Docker
* You have to load balance across different ports.  We don't want to think about ports, let alone track them.
* How do we do this in Kubernetes?  Use services.
* How does it work under the hood?  Look at endpoints.
* Kubernetes uses service discovery to track this on our behalf.  We don't have to worry about this!
* Show dynamic service discovery; changing endpoints.
