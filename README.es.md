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

## 