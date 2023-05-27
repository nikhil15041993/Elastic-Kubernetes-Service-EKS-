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
