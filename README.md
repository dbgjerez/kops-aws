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

