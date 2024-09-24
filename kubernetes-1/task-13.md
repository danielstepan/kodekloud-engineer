# Deploy Highly Available Pods with Replication Controller

The Nautilus DevOps team is establishing a `ReplicationController` to deploy multiple pods for hosting applications that require a highly available infrastructure. Follow the specifications below to create the `ReplicationController`:


Create a `ReplicationController` using the `httpd` image, preferably with `latest` tag, and name it `httpd-replicationcontroller`.

Assign labels `app` as `httpd_app`, and `type` as `front-end`. Ensure the container is named `httpd-container` and set the replica count to `3`.


All `pods` should be running state post-deployment.

## Solution
- Create and apply the following manifest

   ```
    apiVersion: v1
    kind: ReplicationController
    metadata:
    name: httpd-replicationcontroller
    labels:
        app: httpd_app
        type: front-end
    spec:
    replicas: 3
    selector:
        app: httpd_app
        type: front-end
    template:
        metadata:
        name: httpd-replicationcontroller
        labels:
            app: httpd_app
            type: front-end
        spec:
        containers:
        - name: httpd-container
            image: httpd:latest
   ```