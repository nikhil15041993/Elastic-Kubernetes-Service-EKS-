## install stable prometheus-community
```
helm install stable prometheus-community/kube-prometheus-stack -n prometheus -f values.yaml
```

### Step 1: Create an IAM Policy for the EBS CSI Driver

1. **Create a Policy JSON File**:  
   Save the following content as `ebs-csi-policy.json`:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "ec2:AttachVolume",
           "ec2:CreateVolume",
           "ec2:DeleteVolume",
           "ec2:DescribeVolumes",
           "ec2:DescribeVolumeStatus",
           "ec2:DetachVolume",
           "ec2:ModifyVolume",
           "ec2:DescribeSnapshots",
           "ec2:CreateSnapshot",
           "ec2:DeleteSnapshot",
           "ec2:CreateTags",
           "ec2:DescribeInstances"
         ],
         "Resource": "*"
       },
       {
         "Effect": "Allow",
         "Action": "sts:AssumeRole",
         "Resource": "*"
       }
     ]
   }
   ```


 **Create the IAM Policy**:  
Use the following command to create the policy:
```
aws iam create-policy --policy-name AmazonEKS_EBS_CSI_Driver_Policy --policy-document file://ebs-csi-policy.json
```

### Step 2: Create an IAM Role for the EBS CSI Driver
1. **Create a Trust Policy JSON File**:  
   Save the following content as `trust-policy.json`:

```
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/oidc.eks.<REGION>.amazonaws.com/id/<OIDC_ID>"
         },
         "Action": "sts:AssumeRoleWithWebIdentity",
         "Condition": {
           "StringEquals": {
             "oidc.eks.<REGION>.amazonaws.com/id/<OIDC_ID>:sub": "system:serviceaccount:kube-system:ebs-csi-controller-sa"
           }
         }
       }
     ]
   }

```
 **Create the IAM Role**:  
Run the following command to create the role:
```
aws iam create-role --role-name EBSCSIRole --assume-role-policy-document file://trust-policy.json
```   

### Step 3: Attach the IAM Policy to the Role
1. **Attach the Policy**:  
Use the ARN from Step 1 to attach the policy to the IAM role:
```
aws iam attach-role-policy --role-name EBSCSIRole --policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/EBSCSIP
```   


### Alternative: Use `eksctl` to Create a Service Account
You can also use `eksctl` to create a Kubernetes service account with the required IAM role:

```
eksctl create iamserviceaccount \
  --region <YOUR_REGION> \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster <YOUR_CLUSTER_NAME> \
  --attach-policy-arn arn:aws:iam::<YOUR_AWS_ACCOUNT_ID>:policy/AmazonEKS_EBS_CSI_Driver_Policy \
  --approve \
  --override-existing-serviceaccounts
```

### Step 4: Deploy the EBS CSI Driver
1. **Apply the EBS CSI Driver Manifests**:  
   Use the following command to deploy the driver:
```
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/ecr/?ref=release-1.25"
```

2. **Verify the Installation**:  
   Check that the EBS CSI driver pods are running in the `kube-system` namespace:
   ```
   kubectl get pods -n kube-system
   ```

### Step 6: Create Storage Resources

1. **Create the StorageClass**:  
   Save the following as `ebs-storageclass.yaml`:

   ```
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: ebs-sc
   provisioner: ebs.csi.aws.com
   volumeBindingMode: WaitForFirstConsumer
   parameters:
     type: gp2
     fsType: ext4
     encrypted: "false"
   ```

    Apply the StorageClass:

   ```
   kubectl apply -f ebs-storageclass.yaml
   ```

   2. **Create the PersistentVolumeClaim (PVC)**:  
   Save the following content to `ebs-pvc.yaml`:

   ```
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: ebs-pvc
   spec:
     accessModes:
       - ReadWriteOnce
     storageClassName: ebs-sc
     resources:
       requests:
         storage: 10Gi
  
    ```
   Apply the PVC:
   ```
   kubectl apply -f ebs-pvc.yaml
   ```

### Step 7: Resize the EBS Volume
1. **Edit the PVC**:  
   Modify the `ebs-pvc.yaml` file to increase the volume size:

   
   kubectl edit pvc ebs-pvc
