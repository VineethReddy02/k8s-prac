


### ReplicaSet

Selector field in ReplicaSet spec is mandatory compared to replicationcontroller this field is used to consider/validate the existing pods with labels under (selector: matchLabels:)  
and they then create the desired number of replicas. If there are pods with matching labels
replicaset will consider existing pods with mathcing labels either create/terminate as per
required number of replicas.

### Commands

To modify the existing object with latest changes from yml file. Note only mutable fields can
undergo replace operation.
```kubectl replace -f replicaset-definition.yml```

To scale the number replicas without changes in yml file
```kubectl scale --replicas=6 -f rs.yml```

To scale the number of replicas with replicaset name
```kubectl scale --replicas=6 replicaset my-rs-app```


### Deployment

The deployment provides us to upgrade underlying instances seamlessly, undo, pause and resume the
changes. WHen deployment is created replicaset is automatically created.

#### kubectl run

Create an NGINX Pod

```kubectl run --generator=run-pod/v1 nginx --image=nginx```



Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

```kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml```



Create a deployment

```kubectl run --generator=deployment/v1beta1 nginx --image=nginx```



Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

```kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run -o yaml```


Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)

```kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml```



Save it to a file - (If you need to modify or add some other details before actually creating it)

```kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml > nginx-deployment.yaml```

Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379

```kubectl expose pod redis --port=6379 --name redis-service --dry-run -o yaml```

(This will automatically use the pod's labels as selectors)

Or

kubectl create service clusterip redis --tcp=6379:6379 --dry-run -o yaml  (This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)



Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

```kubectl expose pod nginx --port=80 --name nginx-service --dry-run -o yaml```

(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

```kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run -o yaml```

(This will not use the pods labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

### Namespace

Create namespace

```kubectl create namespace nm01```



### Service

Service is my default spanned across the cluster i.e it is available across the cluster. The service to pod traffic is based on randon algorithm. If there multiple instances of a pod. Service distributes the traffic to respective pods randomly. Service takes the pod creation/termination to distribute the traffic.

Service in kubernetes are three types.

1. NodePort: The nodeport service is a service which is binded to one of the ports of the node. With this we can hit the service using node ip + binded port. On nodeport creation clusterip is assinged by default. Node port ip can only be in range of 30000 - 32767. WHen we create nodeport
It accessible from all the nodes i.e node ip + nodeport.

In nodeport we use three terms
    - TargetPort: Target port is exposed port of the pod.
    - Port: Port is the service port. Service is virtual server within the cluster.
    - NodePort: NodePort is the port assigned from one of the port from the node ip. This can only be from 30000 - 32767. 

2. ClusterIP: The cluster is a service which is virtual ip dedictaed to within the cluster use. on creation of a service cluster ip is created by default. The cluster ip is an interface used to group the corresponding pods to loosely couple the pod to pod communication.

3. LoadBalancer: 


### Taints & Tolerations:

Taints & tolerations doesn't guarantee a pod to be scheduled on a particular node.
When we taint & add tolerations to pod. It is not guanranteed that pod is always scheduled on tainted node. It can also be scheduled on other nodes.

```kubectl taint nodes node-name key=value:taint-effect```

taint effect can be either Noschedule or PreferNoSchedule or NoExecute

NoSchedule: This will not let any new schedulings to be scheduled on that node without toleration,

PreferNoSchedule: This will try not scheduling but not guaranteed.

NoExecute: This will not schedule any new objects without tolerations. Also evicts the existing resources which doesn't tolerate to the taint.


### Node Selctors:

Node-selectors helps us in scheduling pods on selcted nodes specfied in pod manifest using the node labels.

```kubectl label nodees <node-name> <label-name>=<label-value>```

Example:
    ```kubectl label nodes node-1 size=Large```

### Node Affinity

Node affinity is used for complex selection operation on multiple nodes like one of, etc...

Available:

requiredDuringSchedulingIgnoredDuringExecution
This make sure the pod with node affinity is only placed on the node with matching label. Else the pod will not be scheduled.

preferredDuringSchedulingIgnoredDuringExecution
This will try scheduling the pod with node which has matching label else it will schedule the pod somewhere else.

For both the above types "IgnoredDuringExecution" means after the pods scheduled. If the labels are chnaged on the node for some reason it will ignore that action.


Note: Combination of node affinity, taints & tolerations will helps us in achieving the right scheduling as desired.

As taints & tolerations always doesn't guanrentee the desired scheduling. node affinity will allow you to achieve desired scheduling but if you want to restrict some of the pods to not to be scheduled on specific nodes we should use taints & tolerations.
Both used together will help in better scheduling.


### Static Pods

Static pods are managed by kubelet. They can be created/deleted by placing/removing the files from ```/etc/kubernetes/manifests``` and they are mirrored to api-server. Just to list them when we hit api-server.
They are mainly used to deploy control plane components.


### Multiple Schedulers

As kubernetes is highly extensible you can have custom schedulers. In HA clusters with multiple masters. We configure ```--leader-elect=true``` to make sure only one scheduler will be scheduling the jobs across cluster.
If we want a pod to be scheduled by custom scheduler. We should specify it in pod manifest as

```
spec:
    schedulerName: my-custom-scheduler
```

### Application lifecycle management

Common cmds to observe rollout

```kubectl rollout status deployment/myapp-deployment```

```kubectl rollout history deployment/myapp-deployment```

```kubectl rollout undo deployment/myapp``` to undo a rollout (This reverst the deployment to previous revision)

#### Deployment Strategy

1. Recreate strategy (This will delete all the existing pods & recreate the newer version of pods this way of rollout will expereience a downtime. Note this is not a default rollout strategy)

2. Rolling Update (This will delete existing pod & create a newer version of pod one by one making sure application is up with zero downtime. This is the default rollout strategy)

### Commands & arguments

If you want to overide the image entrypoint or CMD in pod manifest. We should be using "command" field to overide the entrypoint instruction & if you want to append to entrypoint or overide CMD you should use "args" 

Example:

```
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep2.0"] # This is to overide the entrypoint instruction
      args: ["10"] # This is to either append to entrypoint or to overide the CMD.
```

### Configuring environment varibales

```
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          value: pink
```

1. Normal key/value plain test

```
env:
  - name: APP_COLOR
    value: pink
```

```
env:
  - name: APP_COLOR
    valueFrom:
        configMapKeyRef:
```

```
env:
  - name: APP_COLOR
    valueFrom:
        secretKeyRef:
```

### ConfigMaps

Configmaps are used to manage environments variables in a common location and make them accessible to different pods.

We can inject env variables in three ways.

1. By exposing all the data in a config map.
```
envFrom:
  - configMapRef:
        name: app-config
```

2. Injecting specifc env variable from configmap using its name.
```
env:
  - name: APP_COLOR
    valueFrom:
      configMapRef:
        name: app-config
        key: APP_COLOR
```

3. By exposing config map as volume using a file.
```
volumes:
- name: app-config-volume
  configMap:
    name: app-config
```

### Secrets

Secrets are similar to configmaps but its sensitive data. So we need to hash(encode it base64)
and store them. 

1. By exposing all the data in a secret.
```
envFrom:
  - secretRef:
        name: app-secret
```

2. Injecting specifc secret value from secret using its name.
```
env:
  - name: DB_Password
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: DB_Psssword
```

3. By exposing secret as volume using a file.
```
volumes:
- name: app-secret-volume
  configMap:
    name: app-secret
```

#### Drain

This takes the node down i.e by removing all the pods from it and scheduling them on other nodes
```kubectl drain node-1```

#### Cordan
This makes the node as unschedulable. No new pods will be scheduled on this node.
```kubectl cordan node-2```

#### Uncordon
This will amke the node backup to the k8s cluster. But the removed pods will not automaticlly come on to this node. New pod creation can be scheduled on this node.
```kubectl uncordon node-1```

### Cluster Upgrades

First we should update kubeadm using

```apt-get upgrade -y kubeadm=1.12.0-00```

See all the info of avialble latest version using
```kubectl upgrade plan```

Upgrade the master components.

```kubeadm upgrade apply v1.12.0```

Now kubeadm & kubelet is not upgraded yet. For this you ssh into each node and upgrade. Before we do that. we should drain the node. using

```kubectl drain node-1``` All the pods are scheduled on other nodes. we good to upgrade

```apt-get upgrade -y kubeadm=1.12.0-00```
```apt-get upgrade -y kubelet=1.12.0-00```
```kubeadm upgrade node config --kubelet version v1.12.0```
```systemctl restart kubelet```

### Backup

Backing up all the configuration files in the cluster.

```kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml```

### Security in Kubernetes

 In kubernetes cluster we create a root CA i.e certificate authority.
 Below are the cmds to generate a self signed CA

 ```openssl genrsa -out ca.key 2048```

 ```openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr```

 ```openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt``` // self signing the csr
 
 Below are the cmds to generate public & private keys for admin user using ca crt.

 ```openssl genrsa -out admin.key 2048```

 ```openssl req -new -key admin.key -subj \ "/CN=kube-admin" -out admin.csr```

 ```openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt``` // getting the crt signing by ca. Obtained from above self singed ca.

To extract the info from crt

```openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout```

There is an object called "CertificateSigningRequest" to sign the CSR with the root kubernetes CA.

This gets all the csr pending jobs.
```kubectl get csr```

This is to the sign the certificatesigning request
```kubectl certificate approve jane```

### NetworkPolicy

With the networkpolicy primitive we can retsrict ingress/egress for a specific pod.





### ETCD

Backing up etcd

etcdctl snapshot save snapshot.db

etcdctl snapshot restore snapshot.db --data-dir /var/lib/etcd-from-backup --initial-cluster master-1=https://192.168.5.11:2380 --initial-cluster-token etcd-cluster-1 --initial-advertise-peer-urls https://${INTERNAL_IP}:2380

systemctl daemon-reload

service etcd restart

Create a static pod named static-busybox that uses the busybox image and the command sleep 1000


Weight: 8

Name: static-busybox
Image: busybox



Use JSON PATH query to retrieve the osImages of all the nodes and store it in a file /opt/outputs/nodes_os_x43kj56.txt


The osImages are under the nodeInfo section under status of each node.


Weight: 6

Create a Persistent Volume with the given specification.


Weight: 8

Volume Name: pv-analytics
Storage: 100Mi
Access modes: ReadWriteMany
Host Path: /pv/data-analytics

Create a new pod called super-user-pod with image busybox:1.28. Allow the pod to be able to set system_time


The container should sleep for 4800 seconds


Weight: 8

Pod: super-user-pod
Container Image: busybox:1.28
SYS_TIME capabilities for the conatiner?


Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Record the version. Next upgrade the deployment to version 1.17 using rolling update. Make sure that the version upgrade is recorded in the resource annotation.


Weight: 15

Deployment : nginx-deploy. Image: nginx:1.16
Image: nginx:1.16
Task: Upgrade the version of the deployment to 1:17
Task: Record the changes for the image upgrade


ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --cacert /etc/kubernetes/pki/etcd/ca.crt \
  --cert /etc/kubernetes/pki/etcd/server.crt \
  --key /etc/kubernetes/pki/etcd/server.key