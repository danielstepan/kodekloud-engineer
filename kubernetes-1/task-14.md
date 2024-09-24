# Resolve Volume Mounts Issue in Kubernetes
We encountered an issue with our Nginx and PHP-FPM setup on the Kubernetes cluster this morning, which halted its functionality. Investigate and rectify the issue:

The pod name is `nginx-phpfpm` and configmap name is `nginx-config`. Identify and fix the problem.


Once resolved, copy `/home/thor/index.php` file from the `jump host` to the `nginx-container` within the nginx document root. After this, you should be able to access the website using `Website` button on the top bar.

## Solution
- Explore the content of the configmap:

    ```
    k get cm/nginx-config -o yaml
    ```

    ```
    apiVersion: v1
    data:
    nginx.conf: |
        events {
        }
        http {
        server {
            listen 8099 default_server;
            listen [::]:8099 default_server;

            # Set nginx to serve files from the shared volume!
            root /var/www/html;
            index  index.html index.htm index.php;
            server_name _;
            location / {
            try_files $uri $uri/ =404;
            }
            location ~ \.php$ {
            include fastcgi_params;
            fastcgi_param REQUEST_METHOD $request_method;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_pass 127.0.0.1:9000;
            }
        }
        }
    kind: ConfigMap
    metadata:
    annotations:
        kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","data":{"nginx.conf":"events {\n}\nhttp {\n  server {\n    listen 8099 default_server;\n    listen [::]:8099 default_server;\n\n    # Set nginx to serve files from the shared volume!\n    root /var/www/html;\n    index  index.html index.htm index.php;\n    server_name _;\n    location / {\n      try_files $uri $uri/ =404;\n    }\n    location ~ \\.php$ {\n      include fastcgi_params;\n      fastcgi_param REQUEST_METHOD $request_method;\n      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;\n      fastcgi_pass 127.0.0.1:9000;\n    }\n  }\n}\n"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"nginx-config","namespace":"default"}}
    creationTimestamp: "2024-09-24T18:57:27Z"
    name: nginx-config
    namespace: default
    resourceVersion: "4831"
    uid: 946b7f87-34ac-4e8d-9996-9836216e696d
    ```

- Explore the pod definition:

    ```
    k get pod/nginx-phpfpm -o yaml
    ```
    ```
    apiVersion: v1
    kind: Pod
    metadata:
    annotations:
        kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"php-app"},"name":"nginx-phpfpm","namespace":"default"},"spec":{"containers":[{"image":"php:7.2-fpm-alpine","name":"php-fpm-container","volumeMounts":[{"mountPath":"/var/www/html","name":"shared-files"}]},{"image":"nginx:latest","name":"nginx-container","volumeMounts":[{"mountPath":"/usr/share/nginx/html","name":"shared-files"},{"mountPath":"/etc/nginx/nginx.conf","name":"nginx-config-volume","subPath":"nginx.conf"}]}],"volumes":[{"emptyDir":{},"name":"shared-files"},{"configMap":{"name":"nginx-config"},"name":"nginx-config-volume"}]}}
    creationTimestamp: "2024-09-24T18:57:27Z"
    labels:
        app: php-app
    name: nginx-phpfpm
    namespace: default
    resourceVersion: "4867"
    uid: 970dced7-c909-488f-9623-eb6001241d91
    spec:
    containers:
    - image: php:7.2-fpm-alpine
        imagePullPolicy: IfNotPresent
        name: php-fpm-container
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/www/html
        name: shared-files
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-jlwdx
        readOnly: true
    - image: nginx:latest
        imagePullPolicy: Always
        name: nginx-container
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /usr/share/nginx/html
        name: shared-files
        - mountPath: /etc/nginx/nginx.conf
        name: nginx-config-volume
        subPath: nginx.conf
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-jlwdx
        readOnly: true
    dnsPolicy: ClusterFirst
    enableServiceLinks: true
    nodeName: kodekloud-control-plane
    preemptionPolicy: PreemptLowerPriority
    priority: 0
    restartPolicy: Always
    schedulerName: default-scheduler
    securityContext: {}
    serviceAccount: default
    serviceAccountName: default
    terminationGracePeriodSeconds: 30
    tolerations:
    - effect: NoExecute
        key: node.kubernetes.io/not-ready
        operator: Exists
        tolerationSeconds: 300
    - effect: NoExecute
        key: node.kubernetes.io/unreachable
        operator: Exists
        tolerationSeconds: 300
    volumes:
    - emptyDir: {}
        name: shared-files
    - configMap:
        defaultMode: 420
        name: nginx-config
        name: nginx-config-volume
    - name: kube-api-access-jlwdx
        projected:
        defaultMode: 420
        sources:
        - serviceAccountToken:
            expirationSeconds: 3607
            path: token
        - configMap:
            items:
            - key: ca.crt
                path: ca.crt
            name: kube-root-ca.crt
        - downwardAPI:
            items:
            - fieldRef:
                apiVersion: v1
                fieldPath: metadata.namespace
                path: namespace
    ```
-  We can see that the nginx root location is set to `root /var/www/html;` in the configmap. The nginx container in the pod mounts the shared folder to `/usr/share/nginx/html`, we change the path to `/var/www/html`.

    ```
    k edit pod/nginx-phpfpm 
    ```
    ```
    error: pods "nginx-phpfpm" is invalid
    A copy of your changes has been stored to "/tmp/kubectl-edit-3493646507.yaml"
    error: Edit cancelled, no valid changes were saved.
    ```
    ```
    k apply -f /tmp/kubectl-edit-3493646507.yaml --force
    ```

- At the end we copy the file from the local machine to the remote pod:

    ```
    k cp /home/thor/index.php nginx-phpfpm:/var/www/html/
    Defaulted container "php-fpm-container" out of: php-fpm-container, nginx-container
    ```