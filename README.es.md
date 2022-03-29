# kops-aws
Kops es una herramienta que nos facilita el aprovisionamiento de clúster de Kubernetes en diferentes proveedores cloud. 

En este workshop mostraré cómo se puede realizar dicho despliegue, focalizando en desplegar un pequeño clúster para desarrollo. 

> **AVISO** Para realizar el despliegue en un clúster en alta disponibilidad habría que hacer varios cambios, este caso es sólo para desarrollo.

## Prerequisites
Kops permite el despliegue en diversos proveedores cloud. En este ejemplo vamos a utilizar AWS, por tanto: es importante tener una cuenta de AWS correctamente configurada:

* [Cómo crear usuario admin](https://docs.aws.amazon.com/sdk-for-go/v1/developer-guide/configuring-sdk.html#specifying-credentials) (en caso de no tenerlo)
* [Configuración de credencias para cli de AWS](https://docs.aws.amazon.com/sdk-for-go/v1/developer-guide/configuring-sdk.html#specifying-credentials)

Además, es necesario instalar las siguiente heramientas:

* Kops cli: [Descarga](https://kops.sigs.k8s.io/getting_started/install/)
* AWS cli: [Documentación](https://aws.amazon.com/cli/)

## Proceso

### Usuario Kops
Es recomendable crear un usuario de IAM aws para todo el proceso relacionado con Kops. Este usuario lo llamaramos ```kops```.

EL usuario ```kops``` necesita los siguientes permisos de IAM:

```properties
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
AmazonSQSFullAccess
AmazonEventBridgeFullAccess
```

Para crear el usuario desde linea de comandos podemos usar las siguientes instrucciones: 

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

Una vez el usuario ha sido creado, es importante guardar el  ```SecretAccessKey``` y ```AccessKeyID```. Con dichos parámetros, añade el usuario creado a tu fichero de credencias de AWS (```~/.aws/credentials```).

Asegurate de seguir todo el proceso con el usuario ```kops```.

```zsh
AWS_PROFILE=kops
```

### Domain
Existen muchas formas de configurar el DNS y dominio. La forma más sencilla es teniendo tu propio dominio hospedado en AWS. 

En mi caso, tenía mi propio dominio en AWS desde hace tiempo. Tú puedes elegir la forma que más te convenga del siguiente listado: ([DNS y dominio](https://kops.sigs.k8s.io/getting_started/aws/#configure-dns)).

### Estado
Kops guarda el estado del cluster en un bucket de s3.

Vamos a crear el bucket necesario en la región ```us-east-1``` tal y como recomienda la documentación oficial de Kops.

> **NOTE** Independientemente de la localización del bucket, el cluster podrá ser creado en cualquier región.

Asignamos el nombre del bucket a una variable: 

```zsh
BUCKET_NAME=k8s.dborrego.example
```

Creamos el bucket:

```zsh
aws s3api create-bucket \
    --bucket ${BUCKET_NAME} \
    --region us-east-1
```

Se recomienda activar el versionado del bucket por si se tiene que revertir algún cambio:

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
```