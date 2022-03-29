# kops-aws
A cheap way to deploy a Kubernetes cluster is using Kops which supply all we need.

In this workshop, I will show how we can deploy a little dev cluster. 

> **WARNING** This cluster should be used only to development purpose as doesn't exist high availability. To have a 24/7 available cluster, it is important to change the deployment. 

## Prerequisites
Many cloud providers can be used, in this example we will use AWS so it's mandatory to have an account correctly configured.

* [How to create the IAM admin user](https://docs.aws.amazon.com/sdk-for-go/v1/developer-guide/configuring-sdk.html#specifying-credentials) (if you dont have one)
* [Configure AWS Credentials](https://docs.aws.amazon.com/sdk-for-go/v1/developer-guide/configuring-sdk.html#specifying-credentials)

In addition, we need install the following tools before to continue the workshop:

* Kops cli: [Download](https://kops.sigs.k8s.io/getting_started/install/)
* AWS cli: [Documentation](https://aws.amazon.com/cli/)

## How to

### Kops user
To deploy the cluster, it is recommended to create a IAM AWS user for Kops, who we will call ```kops```.

The ```kops``` user requires the following IAM permissions:

```properties
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
AmazonSQSFullAccess
AmazonEventBridgeFullAccess
```

To create the IAM user from command line: 
```zsh
aws iam create-group --group-name kops

aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonSQSFullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEventBridgeFullAccess --group-name kops

aws iam create-user --user-name kops

aws iam add-user-to-group --user-name kops --group-name kops

aws iam create-access-key --user-name kops
```

After the user is created, you have to record the ```SecretAccessKey``` and ```AccessKeyID```. 

In this point, you should add this user in your aws credentials file and continue the process with the ```kops``` user.

```zsh
AWS_PROFILE=kops
```

### Domain
Kops saves the state of the cluster in a s3

### State