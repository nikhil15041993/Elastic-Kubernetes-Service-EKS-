# Activate Cluster autoscaler

The cluster autoscaler automatically launches additional worker nodes if more resources are needed, and shutdown worker nodes if they are underutilized. The autoscaling works within a nodegroup, hence create a nodegroup first which has this feature enabled.

## create nodegroup with autoscaler enabled

Create a yml file eg: eks-course.yaml 

```
eks-course.yaml
======================


 apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: EKS-course-cluster
  region: us-east-1

nodeGroups:
  - name: scale-east1c
    instanceType: t2.small
    desiredCapacity: 1
    maxSize: 10
    availabilityZones: ["us-east-1c"]
    iam:
      withAddonPolicies:
        autoScaler: true
    labels:
      nodegroup-type: stateful-east1c
      instance-type: onDemand
    ssh: # use existing EC2 key
      publicKeyName: eks-course
      
      
  - name: scale-east1d
    instanceType: t2.small
    desiredCapacity: 1
    maxSize: 10
    availabilityZones: ["us-east-1d"]
    iam:
      withAddonPolicies:
        autoScaler: true
    labels:
      nodegroup-type: stateful-east1d
      instance-type: onDemand
    ssh: # use existing EC2 key
      publicKeyName: eks-course
      
      
  - name: scale-spot
    desiredCapacity: 1
    maxSize: 10
    instancesDistribution:
      instanceTypes: ["t2.small", "t3.small"]
      onDemandBaseCapacity: 0
      onDemandPercentageAboveBaseCapacity: 0
    availabilityZones: ["us-east-1c", "us-east-1d"]
    iam:
      withAddonPolicies:
        autoScaler: true
    labels:
      nodegroup-type: stateless-workload
      instance-type: spot
    ssh: 
      publicKeyName: eks-course

availabilityZones: ["us-east-1c", "us-east-1d"]

```


```bash
eksctl create nodegroup --config-file=eks-course.yaml

```

ckeck the newly created nodes by using

```
kubectl get nodes
```

## deploy the autoscaler

the autoscaler itself:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
```

put required annotation to the deployment:

```bash
kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false" --overwrite
```

get the autoscaler image version:  
open https://github.com/kubernetes/autoscaler/releases and get the latest release version matching your Kubernetes version, e.g. Kubernetes 1.14 => check for 1.14.n where "n" is the latest release version

edit deployment and set your EKS cluster name:

```bash
kubectl -n kube-system edit deployment.apps/cluster-autoscaler
```

* set the image version at property ```image=k8s.gcr.io/cluster-autoscaler:vx.yy.z```  
* set your EKS cluster name at the end of property ```- --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<<EKS cluster 

## view cluster autoscaler logs

```bash
kubectl -n kube-system logs deployment.apps/cluster-autoscaler
```

## test the autoscaler

### create a deployment of nginx

```bash
kubectl apply -f nginx-deployment.yaml
```

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: test-autoscaler
spec:
  replicas: 1
  template:
    metadata:
      labels:
        service: nginx
        app: nginx
    spec:
      containers:
      - image: nginx
        name: test-autoscaler
        resources:
          limits:
            cpu: 300m
            memory: 512Mi
          requests:
            cpu: 300m
            memory: 512Mi
      nodeSelector:
        instance-type: spot
        
```

### create a deployment of nginx

```bash
kubectl apply -f nginx-deployment.yaml
```

### check the pods

```
kubectl get pods
```
### filter the pods by the instance type

```
kubectl get nodes -l instance-type=spot
```


### scale the deployment

```bash
kubectl scale --replicas=3 deployment/test-autoscaler
```

verify the current state

```
kubectl get pods
```
check the node also ```kubectl get nodes -l instance-type=spot```

in our spot node we using t3 small instance so in that instance having 2gb RAM so we can set 3 nginx pods configure on```nginx-deployment.yaml``.
if we increse the repleca as 4 the new node will create and add the pods on it 

```bash
kubectl scale --replicas=4 deployment/test-autoscaler
```
if we reduce the replica as 3 the newly created node instance ) will terminated.



### view cluster autoscaler logs

```bash
kubectl -n kube-system logs deployment.apps/cluster-autoscaler | grep -A5 "Expanding Node Group"

kubectl -n kube-system logs deployment.apps/cluster-autoscaler | grep -A5 "removing node"
```
