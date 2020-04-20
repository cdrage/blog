---
layout: post
---

* TOC
{:toc}


# Notes on the book: Designing Distributed Systems

## Sidecar pattern

Definition: An application made up of two containers (application + side-car). The side-car is normally transparent and helps augment / improve the initial running application container

Tips:
  - Consider implementing a  `/topz` http endpoint to provide a readout of resource usage (all of this on the sidecar container..)
  - Use it for monitoring also, create a /check endpoint or something similar
  - In the example, the sidecar shares the same filesystem and runs a back loop to pull a git repository

## Ambassador pattern

Sits infront of your application container. Essentially a proxy to your application. Can proxy these connections to different pods.

## Adapter pattern

Creating another container that is added to the application's pod of containers as a means of "adapting" components such as monitoring / logging into something consumable. Ex. Redis does not export logs well, so you'll add an adapter for redis logging to be consumed by Prometheus.

Tips:
  - Create a container that checks a database and then outputs "OK" with HTTP code 200 on /health ? Or something similar.

## Replicated load balanced services

- Make sure you deploy a readiness probe when designing / creating replicated services
- Make sure that the same IP address is hitting the same pod / container if you're using cache / cookie / session based instances

## Sharded services

Stateful sets usually used

Shard = only serving a subset of all requests

Great for proxying / cacheing, for example, using memcache in a stateful set to proxy.

Another example: Multiplayer games which have different players from different countries would go to certain servers based on their location.

## Scatter/Gather

Sharded, replicated distributionch

# Kubernetes

Advantages is being able to develop decoupled service-oriented teams that each build a single microservice. Allows mutable infrastructure.

# Bare-metal cluster (at least for development)

**Setup:**

- Deployment: Deploy using [kubeadm-ansible](https://github.com/kairen/kubeadm-ansible)
- Ingress: [ingress-nginx](https://github.com/kubernetes/ingress-nginx)
- TLS: [cert-manager](https://github.com/jetstack/cert-manager) with Let's Encrypt
- LoadBalancer: [metallb](https://github.com/metallb/metallb)
- Storage: [nfs-client](https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client)

## Notes on setting up Let's Encrypt

```sh
helm install \
  --name cert-manager \
  --namespace kube-system \
  --set ingressShim.defaultIssuerName=letsencrypt-staging \
  --set ingressShim.defaultIssuerKind=ClusterIssuer \
  stable/cert-manager \
  --version v0.5.2
```

Setup an issuer:

```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # The ACME server URL
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: "charlie@charliedrage.com"
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-sec-staging
    # Enable HTTP01 validations
    http01: {}
```

Setup ingress:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    certmanager.k8s.io/cluster-issuer: letsencrypt-staging
    kubernetes.io/tls-acme: “true”
    #nginx.ingress.kubernetes.io/rewrite-target: "/"
  name: cockroach-ingress
  namespace: default
spec:
  rules:
    - host: test.charliedrage.com
      http:
        paths:
          - path: /
            backend:
              serviceName: database-cockroachdb-public
              servicePort: 8080
  tls:
    - secretName: tls-staging-cert
      hosts:
        - test.charliedrage.com
```

## LoadBalancer

MetalLB. Provide an ARP range, use `loadbalancerIP` if you want to specify / pick a specific IP!

Have to create ConfigMap before deployment that uses the `metallb-config`.

```yaml
kind: ConfigMap
metadata:
  namespace: default
  name: metallb-config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.1.96-100
```

```sh
helm install --name metallb stable/metallb
```

## Storage

NFS Server + NFS-Client from https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client for dynamic storage provisioning

```sh
# On the host
docker run \
  -d \
  --restart=always \
  --net=host \
  --name nfs \
  --privileged \
  -v /mnt/storage/k8s:/nfsshare \
  -e SHARED_DIRECTORY=/nfsshare \
  cdrage/nfs-server-alpine

# On all servers
sudo apt-get install nfs-common -y 

# For the nfs-client / provisioner
helm install stable/nfs-client-provisioner -n nfs-client --set nfs.server=192.168.1.91 --set nfs.path=/ --set storageClass.defaultClass=true
```


## Monitoring

# Certified Kubernetes Administrator (CKA)

- 100% practical, "fixing" broken environments
- Use lot's of `kubectl`

Curriculum:

- Application Lifecycle Management 8%
- Installation, Configuration & Validation 12%
- Core Concepts 19%
- Networking 11%
- Scheduling 5%
- Security 12%
- Cluster Maintenance 11%
- Logging / Monitoring 5%
- Storage 7%
- Troubleshooting 10%

Course-outline:

- Chapter 1. Course Introduction
- Chapter 2. Basics of Kubernetes
- Chapter 3. Kubernetes Architecture
- Chapter 4. Kubernetes Installation and Configuration
- Chapter 5. Accessing a k8s Cluster and Using the API
- Chapter 6. Replication Controllers and Deployments
- Chapter 7. Volumes and Application Data
- Chapter 8. Services
- Chapter 9. Ingress
- Chapter 10. Additional API Objects
- Chapter 11. Scheduling
- Chapter 12. Logging, Monitoring, and Troubleshooting
- Chapter 13. Third-Party Resources
- Chapter 14. Kubernetes Federation
- Chapter 15. Helm
- Chapter 16. Security

# Components of Kubernetes / Definitions

Internal components:
  - kube-proxy: Routing traffic to load-balanced services in the Kubernetes cluster
  - kube-dns: Provides naming and discovery for the services defined
  - kubernetes-dashboard: GUI for Kubernetes

Most common are:
 - Deployment - stateless persistent apps (http servers, stateless: does not require server to retain session information)
 - StatefulSet - stateful persistent apps (databases, stateful: keeping internet state of the server known)
 - Job - run-to-completion apps (batch jobs)
 - Service - set of Pods with a ClusterIP (networking, accessing the application, using loadbalancer's)

Not-so-common:
 - DaemonSet - ensures that all (or some) nodes run a copy of a Pod
 - CronJob - configuration of a single cronjob (this is in ALPHA)

Manually:
 - Pod - Collection of containers guaranteed to be located on the node and share resources (should be created through Deployment, Job, StatefulSet).
 - ReplicaSet - Same as Replication Controller but has selector support (should be created through Deployment)
 - ReplicationController - Ensures specific number of pod replicas are running at any given time (should be created through Deployment)
 - Container - Only created in the context of a Pod (should be created through Pod)

# Labels and Selectors

Labels / selectors map services and deployments together..

For example:

```yaml
kind: Service
metadata:
  name: redis
  labels:
    app: redis
spec:
  selector:
    app: redis
  ...

# Maps to:
kind: StatefulSet
metadata: 
  name: sharded-redis
spec:
  template:
    metadata:
      labels:
        app: redis
  spec:
    ...

```

Identify and group objects in a Kubernetes cluster.

Update labels on any object:

```sh
# Add
kubectl label pods bar color=red

# Remove
kubectl label pods bar -color
```

Used to map system objects. Example labels:
  - "release" : "stable", "release": "canary"

```json
"labels": {
  "key1" : "value1",
  "key2" : "value2"
}
```

```json
"selector": {
    "component" : "redis"
}
```

# Annotations

Attach non-identifying metadata to objects. So tools or libraries can retrieve it.

Primary use-case would be for rolling deployments. Annoyations are used to track rollout status and provide necessary information required to roll back a deployment.

"namespace" part of the key is important.
examples: `deployment.kubernetes.io/revision` or `kubernetes.io/change-cause`

```json
"annotations": {
  "key1" : "value1",
  "key2" : "value2"
}
```


# Deployments

Deployment manages ReplicaSets and RC's manage Pods.

Deployments main feature is the rolling ability for deploying new appliations.

Strategy is used for example to roll out new software. `Recreate` and `RollingUpdate`.

`Recreate` simply updated the ReplicaSet it manages to use the new image and terminates all the Pods associated. Potentionally catastrophic. Should only be used for TEST environments.

`RollingUpdate` Updates a few Pods at a time, moving incrementally until all the Pods are running the new version of the software.
  - Setting `maxSurge` to 100% is equivilant to a blue/green deployment. The deployment controller scales the new version up to 100% of the old version. Once new version is healthy, it will scale the old version down to 0%.
  - Setting `minReadySeconds` will delay the rollout strategy. The Deployment must wait X seconds after seeing the Pod to become healthy before moving on.
  - Setting the `progressDeadlineSeconds` will set a timeout period for rollout, failing after X seconds and reverting back

Example with using "strategy":

```yaml
...
strategy:
  rollingUpdate:
    # How many extra resources can be created to achieve a rollout. For example, %120 of a 2-container service, so it will scale up to a total of 12 when rolling out 
    maxSurge: 1
    # Set the max "unavailable" during a rolling update
    maxUnavailable: 1
  type: RollingUpdate
...
```

Updating the `change-cause` will trigger a new roll-out:

```yaml
...
spec:
    ...
    template:
        annotations:
            kubernetes.io/change-cause: "Update nginx to 1.9.10"
...
```

You can see revision history (from the change-cause updates) using `kubectl rollout history deployment nginx`

To roll back a previous release: `kubectl rollout undo deployments nginx`


# StatefulSets

Similar to ReplicaSet, but have certain unique properties:
  - Each replica gets a persistent hostname with a unique index (database-0, database-1, etc.)
  - Each replica is created in order from lowest to highest (scaling up)
  - When deleted, each replica will be deleted from order highest to lowest (scaling down)

```yaml
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
    name: mongo
spec:
    serviceName:  "mongo"
    replicas: 3
    template:
        metadata:
            labels:
                app:  mongo
        spec:
            containers:
            - name: mongodb
                image:  mongo:3.4.1
                command:
                - mongod
                - --replSet
                - rs0
                ports:
                - containerPort:  27017
                    name: peer
---
apiVersion: v1
kind: Service
metadata:
    name: mongo
spec:
    ports:
    - port: 27017
        name: peer
    clusterIP:  None
    selector:
        app:  mongo
```

When creating the above example, you'll not only get `mongo.default.svc.cluster.local` as a DNS name, but `mongo-0.mongo.default.svc.cluster.local` and `mongo-1.mongo`. Each of these resolve to the specific IP address of the replica index in the StatefulSet.


# ReplicaSets TODO

# Services

Using services to expose your service (frontend):
- ClusterIP (default, an internal IP)
- NodePort (expose on each Node's IP at a static port)
- LoadBalancer (expose on a cloud provider's load balancer)
- ExternalName (maps the content to an external name, requires kube-dns)

The two ways to expose is publically would be NodePorts or LoadBalancer. ClusterIP is internal and ExternalName is Kubernetes 1.7+ with kube-dns.

## Loadbalancer example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  clusterIP: 10.0.171.239
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 192.0.2.127
```

# Autoscaling / Horizontal Pod Autoscalers

Scaling a replica set based on CPU usage:

```sh
kubectl autoscale rs kuard --min=2 --max=5 --cpu-percent=80
```

Retrieving information:

```sh
kubectl get hpa
```

# DaemonSets

Ensures a copy of a Pod is running across a set of nodes in a Kubernetes cluster

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
    name: fluentd
    namespace:  kube-system
    labels:
        app:  fluentd
spec:
    template:
        metadata:
            labels:
                app:  fluentd
        spec:
            containers:
            - name: fluentd
                image:  fluent/fluentd:v0.14.10
                resources:
                    limits:
                        memory: 200Mi
                    requests:
                        cpu:  100m
                        memory: 200Mi
                volumeMounts:
                - name: varlog
                    mountPath:  /var/log
                - name: varlibdockercontainers
                    mountPath:  /var/lib/docker/containers
                    readOnly: true
            terminationGracePeriodSeconds:  30
            volumes:
            - name: varlog
                hostPath:
                    path: /var/log
            - name: varlibdockercontainers
                hostPath:
                    path: /var/lib/docker/containers
```

Limiting to a specific node:

```sh
kubectl label nodes k0-default-pool-35609c18-z7tb ssd=true
```

```yaml
apiVersion: extensions/v1beta1
kind: "DaemonSet"
metadata:
    labels:
        app:  nginx
        ssd:  "true"
    name: nginx-fast-storage
spec:
    template:
        metadata:
            labels:
                app:  nginx
                ssd:  "true"
        spec:
            nodeSelector:
                ssd:  "true"
            containers:
                - name: nginx
                    image:  nginx:1.10.0
```

# Jobs

Job creates Pods that run until a successful termination (exit with 0). Useful for things such as database migrations or batch jobs.

Job patterns:

| Type                       | Use case                                               | Behavior                                                                           | Completions | Parallelism |
|----------------------------|--------------------------------------------------------|------------------------------------------------------------------------------------|-------------|-------------|
| One shot                   | Database, migrations                                   | Single pod running until completion                                                | 1           | 1           |
| Parallel fixed completions | Multiple pods processing a set of work in parallel     | One or more Pods running one or more times until reaching a fixed completion count | 1+          | 1+          |
| Work queue: parallel jobs  | Multiple pods processing from a centralized work queue | One or more Pods running once until successful termination                         | 1           | 2+          |

One shot:

Job controller is responsible for recreating the Pod until a sucessful termination occurs

```yaml
apiVersion: batch/v1
kind: Job
metadata:
    name: oneshot
    labels:
        chapter:  jobs
spec:
    # To use parallelism / completions
    # parallelism: 5
    # completions: 10
    template:
        metadata:
            labels:
                chapter:  jobs
        spec:
            containers:
            - name: kuard
                image:  gcr.io/kuar-demo/kuard-amd64:1
                imagePullPolicy:  Always
                args:
                - "--keygen-enable"
                - "--keygen-exit-on-complete"
                - "--keygen-num-to-gen=10"
            restartPolicy:  OnFailure
```



# Pods

When you first delete a pod, there is a grace period (default 30 seconds) which allows the pod to complete any pending requests.

Using port forwarding to access a specific pod:

```sh
kubectl port-forward kuard 8080:8080
```


# Namespaces

To set your own context:

```sh
# Using your own namespace
kubectl config set-context my-context --namespace=mystuff

# Set your context
kubectl config use-context my-context
```

# Debugging commands

```sh
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- bash
kubectl cp <pod-name>:/path/to/remote/file /path/to/local/file
```

# Health Checks

Liveness: Checks to see if it works
Readiness: When a container is ready to serve requests

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: kuard
spec:
    containers:
        - image:  gcr.io/kuar-demo/kuard-amd64:1
            name: kuard
            livenessProbe:
                httpGet:
                    path: /healthy
                    port: 8080
                initialDelaySeconds:  5
                timeoutSeconds: 1
                periodSeconds:  10
                failureThreshold: 3
            ports:
                - containerPort:  8080
                    name: http
                    protocol: TCP
```



# Resources

```yaml
...
resources:
  requests:
      cpu:  "500m"
      memory: "128Mi"
...
```


# Endpoints / Service Discovery

Service DNS example:
```
alpaca-prod.default.svc.cluster.local.
```

alpaca-prod: name of the service
default: namespace
svc: recognizing it is a service
cluster.local: it's part of the cluster

Cluster IP addresses change. Endpoints are created in order to track said change. For every Service object, Kubernetes creates an Endpoints object that contains the IP addresses for that service.

Although based on an older implementation, there are certain environment variables that are injected into the pod during start-up, ex:

```
# The ones mainly used
ALPACA_PROD_SERVICE_HOST 10.115.245.13
ALPACA_PROD_SERVICE_PORT 8080

# Other
ALPACA_PROD_PORT tcp://10.115.245.13:8080
ALPACA_PROD_PORT_8080_TCP tcp://10.115.245.13:8080
ALPACA_PROD_PORT_8080_TCP_ADDR 10.115.245.13
ALPACA_PROD_PORT_8080_TCP_PORT 8080
ALPACA_PROD_PORT_8080_TCP_PROTO tcp
```

Custom service discovery / DNS name:

```yaml
kind: Service
apiVersion: v1
metadata:
    name: external-database
spec:
    type: ExternalName
    externalName: "database.company.com
```

This will automatically resolve queries to database.svc.default.cluster to database.company.com.

Mapping to an IP address:

```yaml
---
kind: Service
apiVersion: v1
metadata:
    name: external-ip-database
---
kind: Endpoints
apiVersion: v1
metadata:
    name: external-ip-database
subsets:
    - addresses:
        - ip: 192.168.0.1
        ports:
        - port: 3306
```


# ConfigMaps & Secrets

ConfigMaps:

ConfigMaps allows variables to be defined in a different file

Example using ConfigMaps from configMap as well as a volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: kuard-config
spec:
    containers:
        - name: test-container
            image:  gcr.io/kuar-demo/kuard-amd64:1
            imagePullPolicy:  Always
            command:
                - "/kuard"
                - "$(EXTRA_PARAM)"
            env:
                - name: ANOTHER_PARAM
                    valueFrom:
                        configMapKeyRef:
                            name: my-config
                            key:  another-param
                - name: EXTRA_PARAM
                    valueFrom:
                        configMapKeyRef:
                            name: my-config
                            key:  extra-param
            volumeMounts:
                - name: config-volume
                    mountPath:  /config
    volumes:
        - name: config-volume
            configMap:
                name: my-config
    restartPolicy:  Never
```

Secrets:

Base64 encoded secrets. Can be used within a container through volume.

```json
"volumes": [{
  "name": "foo",
  "secret": {
    "secretName": "mysecret"
  }
}]
```

Create a secret through CLI:

```sh
$ echo -n "admin" > ./username.txt
$ echo -n "1f2d1e2e67df" > ./password.txt
$ kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt
```

Or YAML:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

Example of using a secret volume:

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: dotfile-secret
data:
  .secret-file: dmFsdWUtMg0KDQo=
---
kind: Pod
apiVersion: v1
metadata:
  name: secret-dotfiles-pod
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: dotfile-secret
  containers:
  - name: dotfile-test-container
    image: gcr.io/google_containers/busybox
    command:
    - ls
    - "-l"
    - "/etc/secret-volume"
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```

# Ingresses

Soo... Essentially the absolute end-point to access services publically.

For GKE, this is setup automatically (LoadBalancer). However, for bare-metal, you'll have to use Traefik (which is pretty easy).

This only supports http.

Here's an example:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world
  namespace: default
  #  annotations:
  #  traefik.frontend.rule.type: PathPrefixStrip
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /
        backend:
          serviceName: foobar-gitea
          servicePort: 80
```

# Storage

Creating a PV with NFS:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
    name: foobar1
    labels:
        volume: foobar1
spec:
    accessModes:
      - ReadWriteOnce
      - ReadWriteMany
    capacity:
        storage:  50Gi
    mountOptions:
      - hard
      - nfsvers=4.1
    nfs:
        server: 192.168.7.177
        path: "/k8s/foobar1/"
    persistentVolumeReclaimPolicy: Recycle
```

Creating a MYSQL with storage:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
    name: database
    labels:
        volume: my-volume
spec:
    capacity:
        storage:  1Gi
    nfs:
        server: 192.168.0.1
        path: "/exports"
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
    name: database
spec:
    resources:
        requests:
            storage:  1Gi
    selector:
        matchLabels:
            volume: my-volume
---
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
    name: mysql
    # labels  so  that  we  can bind  a Service to  this  Pod
    labels:
        app:  mysql
spec:
    replicas: 1
    selector:
        matchLabels:
            app:  mysql
    template:
        metadata:
            labels:
                app:  mysql
        spec:
            containers:
            - name: database
                image:  mysql
                resources:
                    requests:
                        cpu:  1
                        memory: 2Gi
                env:
                # Environment variables are not a best  practice  for security,
                # but we're using them  here  for brevity in  the example.
                # See Chapter 11  for better  options.
                - name: MYSQL_ROOT_PASSWORD
                    value:  some-password-here
                livenessProbe:
                    tcpSocket:
                        port: 3306
                ports:
                - containerPort:  3306
                volumeMounts:
                    - name: database
                        # /var/lib/mysql  is  where MySQL stores  its databases
                        mountPath:  "/var/lib/mysql"
            volumes:
            - name: database
                persistentVolumeClaim:
                    claimName:  database
---
apiVersion: v1
kind: Service
metadata:
    name: mysql
spec:
    ports:
    - port: 3306
        protocol: TCP
    selector:
        app:  mysql
```

Creating Dynamic Volume Provisioning:

```yaml
apiVersion: storage.k8s.io/v1beta1
kind: StorageClass
metadata:
    name: default
    annotations:
        storageclass.beta.kubernetes.io/is-default-class: "true"
    labels:
        kubernetes.io/cluster-service:  "true"
provisioner:  kubernetes.io/azure-disk
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
    name: my-claim
    annotations:
        # This links it to the above StorageClass  
        volume.beta.kubernetes.io/storage-class:  default
spec:
    accessModes:
    - ReadWriteOnce
    resources:
        requests:
            storage:  10Gi
```

A Storage Class will automatically create disk objects, in this case, on the Microsoft Azure Platform.



# Volumes

Different ways of using volumes with pods (recommended patterns):
  - Communication / synchronization: Syncing between two containers, for example a Git location + site.
  - Cache: using cache (emptyDir)
  - Persistent data
  - Mounting the host filesystem: For example, access block-level access to a device on the host

```yaml
volumes:
  - name: "kuard-data"
      hostPath:
          path: "/var/lib/kuard"
```

https://kubernetes.io/docs/concepts/storage/volumes/


# CKAD Questions / studying

https://medium.com/bb-tutorials-and-thoughts/practice-enough-with-these-questions-for-the-ckad-exam-2f42d1228552


## Core concepts

```sh
# Create an NGINX pod in the default namespace
kubectl run nginx --image nginx --namespace default --restart Never

# Create and expose 80
kubectl run nginx --image nginx:1.17.4 --restart Never --port 80

# Create a busybox pod with command sleep 3600
kubectl run busybox --image busybox --restart -- /bin/sh -c "sleep 3600"

# Change the image of the pod
kubectl set image pod/nginx nginx=nginx:1.15-alpine

# Go into a pod
kubectl exec -it busybox /bin/sh
```



# Examples

## Parse server

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
    name: parse-server
    namespace:  default
spec:
    replicas: 1
    template:
        metadata:
            labels:
                run:  parse-server
        spec:
            containers:
            - name: parse-server
                image:  ${DOCKER_USER}/parse-server
                env:
                - name: DATABASE_URI
                    value:  "mongodb://mongo-0.mongo:27017,\
                        mongo-1.mongo:27017,mongo-2.mongo\
                        :27017/dev?replicaSet=rs0"
                - name: APP_ID
                    value:  my-app-id
                - name: MASTER_KEY
                    value:  my-master-key
---
apiVersion: v1
kind: Service
metadata:
    name: parse-server
    namespace:  default
spec:
    ports:
    - port: 1337
        protocol: TCP
        targetPort: 1337
    selector:
        run:  parse-server
```

## Ghost

Use the configuration file

```javascript
var path  = require('path'),
        config;
config  = {
        development:  {
                url:  'http://localhost:2368',
                database: {
                        client: 'sqlite3',
                        connection: {
                                filename: path.join(process.env.GHOST_CONTENT,
                                                                        '/data/ghost-dev.db')
                        },
                        debug:  false
                },
                server: {
                        host: '0.0.0.0',
                        port: '2368'
                },
                paths:  {
                        contentPath:  path.join(process.env.GHOST_CONTENT,  '/')
                }
        
```

Configure a configMap

```sh
$ kubectl apply cm  --from-file ghost-config.js ghost-config
```

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
    name: ghost
spec:
    replicas: 1   selector:
        matchLabels:
            run:  ghost
    template:
        metadata:
            labels:
                run:  ghost
        spec:
            containers:
            - image:  ghost
                name: ghost
                command:
                - sh
                - -c
                - cp  /ghost-config/config.js /var/lib/ghost/config.js
                    &&  /entrypoint.sh  npm start
                volumeMounts:
                - mountPath:  /ghost-config
                    name: config
            volumes:
            - name: config
                configMap:
                    defaultMode:  420
                    name: ghost-config
```

Expose "Ghost":

```sh
$ kubectl expose  deployments ghost --port=2368
```

Expose / access with proxy:

```sh
$ kubectl proxy
```

## Redis


