# Create IAM user and permissions for EKS

your IAM user needs to have certain privileges to e.g. create all the required resources and objects.  According AWS Best Practices you should *never* use your root account for working with AWS services.

 

There are 2 attempts to follow:


### 1. provide admin access  


* login with an admin of your AWS account
* go to "IAM" => "users" => click on your user => "Permissions" => "Add permission" => then search for _AdministratorAccess_ and attach this policy  
* Basically your user just requires *one* policy being attached
  - AdministratorAccess  


### 2. provide a dedicated list of privileges/policies  

to cover all the required privileges, first you have to create additional policies

 
EKS-Admin-policy:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "eks:*"
            ],
            "Resource": "*"
        }
    ]
}
```

CloudFormation-Admin-policy:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:*"
            ],
            "Resource": "*"
        }
    ]
}
```

finally, assign the following policies to your IAM user you are going to use throughout the course:
```
  - AmazonEC2FullAccess
  - IAMFullAccess
  - AmazonVPCFullAccess
  - CloudFormation-Admin-policy
  - EKS-Admin-policy  
```
where the last 2 policies are the ones you created above
