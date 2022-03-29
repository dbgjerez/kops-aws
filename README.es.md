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
* Kubectl: [Download](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

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
Como amante de la IaC y GitOps, vamos a realizar el despliegue de forma declarativa. 

En primer lugar, voy a usar una variable para el nombre del cluster:

```zsh
NAME=k8s.dbgjerez.es
```

Ahora la definción del cluster la podemos generar con el siguiente comando: 

```zsh
kops create cluster ${NAME} \                   
    --zones=eu-west-3b \
    --discovery-store=s3://${BUCKET_NAME}/${NAME}/discovery \
    --dry-run \
    -o yaml > $NAME.yaml
```

El fichero ```k8s.dbgjerez.es.yaml``` contrendrá la definición de nuestro cluster. El CRD ```Cluster``` para la defición del cluster y dos CRD  ```InstanceGroup```, uno para los maestros y otros para los workers.

En mi caso voy a modificar el tipo de instancia ec2. Al ser un cluster de desarrollo asumiré el riesgo de utilizar instancias de tipo stop. 

```yaml
machineType: t2.small
maxPrice: "0.008"
maxSize: 1
minSize: 1
```

Con esta configuración, el precio total del cluster durante un mes sería en torno a 10€ a fecha de hoy.

Además de los cambios realizados a nivel de instancias, voy a realizar modificaciones en la definición del cluster para activas los add-ons ```metrics-server``` y ```certManager```. 

```yaml
certManager:
    enabled: true
metricsServer:
    enabled: true
    insecure: false
```

En este punto, tenemos la definición deseada. Ahora solo nos queda aplicarla. 

### Deploy

The following step is deploy the cluster. We will create it applying the files:

```zsh
```