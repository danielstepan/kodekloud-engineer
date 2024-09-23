# Deploy Pods in Kubernetes Cluster
The Nautilus DevOps team is diving into Kubernetes for application management. One team member has a task to create a pod according to the details below:


1. Create a pod named `pod-httpd` using the `httpd` image with the `latest` tag. Ensure to specify the tag as `nginx:latest`.

2. Set the `app` label to `httpd_app`, and name the container as `httpd-container`.

## Solution
- Create a basic pod manifest.

    ```
    k run pod-httpd --image=httpd:latest --dry-run=client -o yaml > pod.yaml
    ```
- Edit the manifest by adding the labels and naming the container.

    ```
    vi pod.yaml
   ```
   ```
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        app: httpd_app
    name: pod-httpd
    spec:
    containers:
    - image: httpd:latest
        name: httpd-container
        resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}     
    ```
    ```
    k apply -f pod.yaml
    ```
