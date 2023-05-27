# Creating and managing clusters


Create a simple cluster with the following command:

```
eksctl create cluster
```
That will create an EKS cluster in your default region (as specified by your AWS CLI configuration) with one managed nodegroup containing two m5.large nodes.

After the cluster has been created, the appropriate kubernetes configuration will be added to your kubeconfig file. This is, the file that you have configured in the environment variable KUBECONFIG or ~/.kube/config by default. The path to the kubeconfig file can be overridden using the --kubeconfig flag.

Using Config Files
You can create a cluster using a config file instead of flags.

First, create ```cluster.yaml``` file:

Example 1:
```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: basic-cluster
  region: eu-north-1

nodeGroups:
  - name: ng-1
    instanceType: m5.large
    desiredCapacity: 10
    volumeSize: 80
    ssh:
      allow: true # will use ~/.ssh/id_rsa.pub as the default ssh key
  - name: ng-2
    instanceType: m5.xlarge
    desiredCapacity: 2
    volumeSize: 100
    ssh:
      publicKeyPath: ~/.ssh/ec2_id_rsa.pub
```


Example 2:

```

apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: EKS-course-cluster
  region: ap-south-1

nodeGroups:
  - name: ng-1
    instanceType: t2.small
    desiredCapacity: 3
    ssh: # use existing EC2 key
      publicKeyName: wazuh-key
```

Next, run this command:

```
eksctl create cluster -f cluster.yaml
```
This will create a cluster as described.

eksctl also creates the config file for _kubectl_. This means we can immediately fire up a check like:

```
kubectl get nodes
```
If you needed to use an existing VPC, you can use a config file like this:

```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: cluster-in-existing-vpc
  region: eu-north-1

vpc:
  subnets:
    private:
      eu-north-1a: { id: subnet-0ff156e0c4a6d300c }
      eu-north-1b: { id: subnet-0549cdab573695c03 }
      eu-north-1c: { id: subnet-0426fb4a607393184 }

nodeGroups:
  - name: ng-1-workers
    labels: { role: workers }
    instanceType: m5.xlarge
    desiredCapacity: 10
    privateNetworking: true
  - name: ng-2-builders
    labels: { role: builders }
    instanceType: m5.2xlarge
    desiredCapacity: 2
    privateNetworking: true
    iam:
      withAddonPolicies:
        imageBuilder: true
```


To delete this cluster, run:
```
eksctl delete cluster -f cluster.yaml
```
