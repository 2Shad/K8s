# Kubernetes (aka K8s)
kubernetes is an open-source container orchestration system for automating software deployment, scaling, and management. Cloud Native Computing Foundation now maintains the project.

![Diagram](diagram.png)

### K8s manages pods in a cluster
- K8s uses defined yaml file for the desired state for a K8 cluster.
- Pods can contain one or multiple containers
- it self manages the pods around the cluster and balances load across the cluster
- if a pod dies, the K8s master respawns a new one to replace it.
- if a worker node crashes, the K8s master spreads the pods that was in that worker node across the rest of the node to meet the required state.
- if a release has critical issues, K8s can automatically roll back its pods to a previous version, this is defined in the yaml file if the feature is required.

### Use cases
- K8s is often used for micro-services, as it eases the orchastration of complicated apps.
- It facilitates moving on-prem apps to cloud.
- It facilitates frequent releases when using CI/CD tools.


## Kubernetes

### Install kubernetes(kubectl) and minikube
- Make sure docker is proparly installed and working.
- `curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb`
- `sudo dpkg -i minikube_latest_amd64.deb`
- `curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"`
- `sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl`
- Check if kubectl is working `kubectl version --client`
- Now switch to root, and do `apt install conntrack`
- While in root start minikube with `minikube start --vm-driver=none`
- Check with `kubectl get all`

### Kubernetes yaml file
- All code is camel case, make sure the tricky syntax is right always refer to docs

#### Basic nginx deployment with 3 replicas
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: 2shad/shadman_nginx
          ports:
            - containerPort: 80
          imagePullPolicy: Always
```

#### Expose the pods with a **NodePort** service
```
apiVersion: v1
kind: Service
metadata:
  name: sev
spec:
  ports:
  - nodePort: 30001
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: NodePort
```

#### Kubernetes Persistent Volumes
Just like docker kubernetes will not persist any data between deployments, therefore for stateful containers, we need persistent volumes to persist our data.

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: db-volume
spec:
  capacity:
    storage: 512Mi
  accessModes:
    - ReadWriteOnce
  storageClassName: hostpath
  hostPath:
    path: "/home/ubuntu/db-volume"
```

#### Kubernetes Persistent Volume Claim
We can make a Persistent Volume Claim on a Volume to claim storage from a Volume in order to associated to a deployment.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-claim
  namespace: default
spec:
  resources:
    requests:
      storage: 256Mi
  storageClassName: hostpath
  accessModes:
    - ReadWriteOnce
```

In order to attach the PVC to a deployment, we need to add **volumeMounts** indented under containers, and **volumes** indented at the same level as containers.

```
          volumeMounts:
            - name:  data
              mountPath: /data/db
      volumes:
      - name:  data
        persistentVolumeClaim:
          claimName: db-claim
```

### Kubernetes in AWS
We are going to use the same minikube in AWS, making sure we use a t2.medium instance, and in our security group we have Ports 30000-32767 open for NodePorts.

#### Kubernetes EBS Persistent volume
We can also use cloud storage for kubernetes persistent volumes, in here we are use AWS Elastic Block Storage as an example, It is essentially a hard drive, and we formatted to ext4.
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: db-volume
  labels:
    type amazonEBS
spec:
  capacity:
    storage: 512Mi
  accessModes:
    - ReadWriteOnce
  awsElasticBlockStore:
    volumeID: vol-bh45lk344b3ui24b
    fsType: ext4
```
Similarly we can make persistent volume claims from this cloud volume for our deployments to use.
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-claim
  namespace: default
  labels:
    type amazonEBS
spec:
  resources:
    requests:
      storage: 256Mi
  accessModes:
    - ReadWriteOnce
  selector:
    matchLabels:
      type: "amazonEBS"
```