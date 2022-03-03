# Kuberneties (aka K8s)
Kubernetes is an open-source container orchestration system for automating software deployment, scaling, and management. Cloud Native Computing Foundation now maintains the project.

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