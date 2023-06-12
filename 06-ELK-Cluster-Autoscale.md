# ELK Cluster Autoscale
 

The cluster autoscaler automatically launches additional worker nodes if more resources are needed, and shutdown worker nodes if they are underutilized. The autoscaling works within a nodegroup, hence create a nodegroup first which has this feature enabled.

 create nodegroup with autoscaler enabled
 
 ```
eksctl create nodegroup --config-file=my-eks.yaml

eksctl delete nodegroup --cluster=MY-EKS-cluster --name=ng-1 --approve
```

my-eks.yaml
```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: MY-EKS-cluster
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

### Deploy the autoscaler

the autoscaler itself:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
```
put required annotation to the deployment:
```
kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false"
```
### get the autoscaler image version

open https://github.com/kubernetes/autoscaler/releases  and get the latest release version matching your Kubernetes version,
e.g. Kubernetes 1.14 => check for 1.14.n where "n" is the latest release version

then edit deployment and set your EKS cluster name:

```
kubectl -n kube-system edit deployment.apps/cluster-autoscaler
```

* set the image version at property   image=k8s.gcr.io/cluster-autoscaler:vx.yy.z 
* set your EKS cluster name at the end of property   - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<<EKS cluster name>> 
  
### view cluster autoscaler logs
  
  ```
  kubectl -n kube-system logs deployment.apps/cluster-autoscaler
  ```
  
### Test the autoscaler

create a deployment of nginx

 ```
 kubectl apply -f nginx-deployment.yaml
 ```
 nginx-deployment.yaml
 ```
 apiVersion: extensions/v1beta1
 apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-container
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
   matchLabels:
     app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx-container
        resources:
          limits:
            cpu: 300m
            memory: 512Mi
          requests:
            cpu: 300m
            memory: 512Mi
      nodeSelector:
        instance-type: ng-mixed1

 ```
 to chek if the pods is running 
 ```
 kubectl get pods
 ```
 to view our spot instance details which th pod is running
 ```
 kubectl get nodes -l instance-type=spot
 ```
 Next we are going to scale the pods in this instance from 1 to 3 
 ```
 kubectl scale --replicas=3 deployment/test-autoscaler
 ```
 to check the pods 

 ``` kubectl get pods -o wide --watch ```

 Here we can see that the pods are incremented from 1 to 3 and all poda are in running state.
 
To test the auto scaler we need to incress the pods size. if the instance have above its capacity it will create a new instance (node).
 
 ```
 kubectl scale --replicas=4 deployment/test-autoscaler
 ```
 to view our spot instance details which th pod is running
 ```
 kubectl get nodes -l instance-type=spot
 ```
 We can see that now we have 2 spot instance. because in on2 spot instance the maxium pod capacity is 3.

If we reduce the number of replica the newly creted instance will terminated.  

view cluster autoscaler logs
 
```
kubectl -n kube-system logs deployment.apps/cluster-autoscaler | grep -A5 "Expanding Node Group"

kubectl -n kube-system logs deployment.apps/cluster-autoscaler | grep -A5 "removing node"
 ```
