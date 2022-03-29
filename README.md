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
* Kubectl: [Download](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

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

In this point, you should add this user in your aws credentials file (```~/.aws/credentials```), and continue the process with the ```kops``` user.

```zsh
AWS_PROFILE=kops
```

### Domain
Exists a lot of ways to configure DNS and domain. I had my own domain in AWS. 

You can choose your best way in this list ([DNS and domain](https://kops.sigs.k8s.io/getting_started/aws/#configure-dns)), even though to buy your own domain in AWS it's the easiest. 

### State
Kops saves the state of the cluster in a s3 bucket. 

We will create a bucket in ```us-east-1``` as Kops official documentation recommends. 

> **NOTE**  You will be able to create the cluster in any region independent of the s3 location.

I will use a variable for the backet prefix name: 

```zsh
BUCKET_NAME=k8s.dborrego.example
```

And now, I create the bucket: 

```zsh
aws s3api create-bucket \
    --bucket ${BUCKET_NAME} \
    --region us-east-1
```

In addition, it is recommend versiong you s3 bucket: 

```zsh
aws s3api put-bucket-versioning \
    --bucket ${BUCKET_NAME}  \
    --versioning-configuration Status=Enabled
```

### Declarative cluster
As a IaC and GitOps lovers, we will deploy our cluster in declarative way. 

Firstly, we will declare an variable with the cluster name:

```zsh
NAME=k8s.dbgjerez.es
```

To get the definition of the cluster: 

```zsh
kops create cluster ${NAME} \                   
    --zones=eu-west-3b \
    --discovery-store=s3://${BUCKET_NAME}/${NAME}/discovery \
    --dry-run \
    -o yaml > $NAME.yaml
```

Now we can open the ```k8s.dbgjerez.es.yaml``` file. This file contains a ```Cluster``` definition and two ```InstanceGroup``` definition, one for master nodes and another for workers. 

In my case, I will modify the ec2 instance type. As a development cluster, I can assume the risk of use spot instances.

```yaml
machineType: t2.small
maxPrice: "0.008"
maxSize: 1
minSize: 1
```

I will use a small instances, one for master and one for workers, so the total monthly price turn around €10.

In addition, I will change some parameters in the cluster definition. Specifically, I will activate the ```metrics-server``` and ```certManager``` add-ons. 

```yaml
certManager:
    enabled: true
metricsServer:
    enabled: true
    insecure: false
```

Now I have the configuration that I like for this workshop. 

### Deploy

The following step is deploy the cluster. We will create it applying the files:

```zsh
❯ kops create -f k8s.dbgjerez.es.yaml

Created cluster/k8s.dbgjerez.es
Created instancegroup/master-eu-west-3b
Created instancegroup/nodes-eu-west-3b

To deploy these resources, run: kops update cluster --name k8s.dbgjerez.es --yes
```

Once the CRDs have been applied, it's time to create the cluster: 

```zsh
❯ kops update cluster --name ${NAME} --yes
I0328 15:05:12.590601   99167 executor.go:111] Tasks: 0 done / 97 total; 49 can run
```

We should wait a few minutes. The following step is login in our cluster:

```zsh
❯ kops export kubecfg --admin
Using cluster from kubectl context: k8s.dbgjerez.es

kOps has set your kubectl context to k8s.dbgjerez.es
```

We are logged in our cluster. If we use the kubectl command we can work with the cluster.

### Check

We will check the nodes: 

```zsh
❯ kubectl get nodes
NAME                                          STATUS     ROLES                              AGE   VERSION
ip-172-20-35-30.eu-west-3.compute.internal    NotReady   node,spot-worker                   24s   v1.23.5
ip-172-20-57-255.eu-west-3.compute.internal   Ready      control-plane,master,spot-worker   97s   v1.23.5
```

And to check the CPU and memory usage:  
```zsh
❯ kubectl top nodes
NAME                                          CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
ip-172-20-39-66.eu-west-3.compute.internal    63m          6%     1003Mi          53%       
ip-172-20-41-136.eu-west-3.compute.internal   120m         12%    1375Mi          73%       
```

### Update the configuration
Thanks we are using the declarative way, to update the cluster we only have to modify the file and apply it: 

```zsh
kops replace -f $NAME.yaml
kops update cluster $NAME --yes
kops rolling-update cluster $NAME --yes
```

## Documentation
* [Kops getting started](https://kops.sigs.k8s.io/getting_started/aws/)
* [Customizing manifests](https://kops.sigs.k8s.io/manifests_and_customizing_via_api/#using-a-manifest-to-manage-kops-clusters)
* [Instance groups](https://kops.sigs.k8s.io/tutorial/working-with-instancegroups/#converting-an-instance-group-to-use-spot-instances)
* [Addons](https://kops.sigs.k8s.io/addons/)