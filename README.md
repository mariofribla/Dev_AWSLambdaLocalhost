# Creando Ambiente LocalHost de AWS Lambda

## INDICE

* Requisitos.
* Configurando Cliente AWSCli.
* Proyecto Lambda.
 * Crear un Proyecto.
* Nuestro Código.
 * serverless.yml
 * Handley.py 
* Probando nuestro Servicio Lambda.
* Ultimo detalles.
* Otros Simuladores.
 * Soporte AWS Lambda S3.
  * serverless.yml
  * Handley.py 
 * Soporte AWS CloudFront Edge Lambda.
  * serverless.yml
  * Handley.py 
 * Soporte Acceso a MySQL desde AWS Python Lambda
  * serverless.yml
  * Handley.py 
 * Instalando Cloud 9.
 * Instalando DynamoDB Localmente con Docker.


## Requisitos

* Contenedor **Docker** versión 18 o superior.
* Ambiente **NodeJs** versión 10 o superior.
* Navegador **firefox** o similar.
* FrameWork **Serverless**. 
* Cliente **AWSCli**.
* Editor **Visual Studio Code** o similar.

### Configurando Cliente AWSCli
```sh
$ aws configure --profile s3local
AWS Access Key ID = S3RVER
AWS Secret Access Key = S3RVER
Default region name [None]:
Default output format [None]:
$
```

## Para cada Proyecto Lambda:

### Crear Proyecto
```sh
$ mkdir -p PrjLambdaWiki/S3/locals3-bucket
$ cd PrjLambdaWiki/
$ serverless create --template aws-python3 --name lambdawiki
$ code .
```
- En el Editor VSCode:
- Edite los archivo **handler.py** y **serverless.yml**.
- Del archivo **serverless.yml** eliminar todos los comentarios(**#**) de este archivo.

- Iniciando con Simulación
```sh
$ npm init -y
$ npm install request --save
```

Instalando librerías Básicas en tu Proyecto Lambda
- Soporte AWS Lambda ApiGateWay:
```sh
$ serverless plugin install -n serverless-plugin-simulate
$ serverless plugin install -n serverless-python-requirements
```

### Nuestro Código
- **serverless.yml**

```yml
service: lambdawiki
provider:
  name: aws
  runtime: python3.8

functions:
  hello:
    handler: handler.hello
    events:
      - http:
          path: test/hello
          method: get
plugins:
  - serverless-plugin-simulate
  - serverless-python-requirements
```

- **handler.py**
```py
import json

def hello(event, context):
    try:
        return dict(
            statusCode=200,
            body="Saludos desde PrjLambdaWiki, Lambda Simulate..."
        )
    except Exception as e:
        return dict(
            statusCode=500,
            body=str(e)
        )
```

## Probando nuestro Servicio Lambda
Para esta prueba necesitamos tener 3 terminales abiertos simultaneamente.
- **Terminal 1**
```sh
$ cd PrjLambdaWiki/
$ code .
```
- **Terminal 2** (Simulador de Lambda)
```sh
$ cd PrjLambdaWiki/
$ serverless simulate lambda -p 4000
Serverless: Starting registry with db at /PrjLambdaWiki/.sls-simulate-registry
Serverless: Starting registry at: http://localhost:4000
```
- **Terminal 3** (Simulador de ApiGateway)
```sh
$ cd PrjLambdaWiki/
$ serverless simulate apigateway -p 5000 --lambda-port 4000
Serverless: Registering 1 functions with http://localhost:4000/functions
Serverless: [GET /test/hello] => ?:hello
Serverless: Invoke URL: http://localhost:5000
```
- **Terminal 1** (Probar Servicio)
```sh
$ cd PrjLambdaWiki/
$ firefox http://localhost:5000
$ firefox http://localhost:5000/test/hello
```
**Nota**: Al acceder la **1era** ves a la URL http://localhost:5000/test/hello, este ejecución se demorará dado que esta creado una imagen Docker como compilando el código para la prestación del servicio. Las siguientes cambios o ejecuciones serán mejor tiempo. Cada ves que se cambie el lenguaje de codificación, la 1era ves pasará esto.

### Ultimo detalles:
El archivo **.gitignore** debe agregar las siguientes lineas:
```txt
# Distribution / packaging
.
.
.
# Serverless directories
.serverless
node_modules
.sls-simulate*
package-lock.json
```

# Otros Simuladores:
### Soporte AWS Lambda S3:
```sh
$ npm install serverless-s3-local --save-dev
$ serverless plugin install -n serverless-s3-local
```
Ejemplo de Configuración
- **handler.yml**
```yml
service: lambdawiki
provider:
  name: aws
  runtime: python3.8

functions:
  hello:
    handler: handler.hello
    events:
      - http:
          path: test/hello
          method: get
  s3hook:
    handler: handler.s3hook
    events:
      - s3: localS3-bucket
custom:
  s3:
    # Port donde se accederá en el localhost al S3 Local.
    port: 9090
    # Definir la ruta correcta del parametro direcotory
    directory: /tmp/localS3-bucket

resources:
  Resources:
    NewResource:
      Type: AWS::S3::Bucket
      Properties:
        BucketName: localS3-bucket

plugins:
  - serverless-plugin-simulate
  - serverless-python-requirements
  - serverless-s3-local
  - serverless-offline
```
- **handler.yml**
```py
import json
from collections import OrderedDict
from pathlib import Path

def hello(event, context):
    try:
        return dict(
            statusCode=200,
            body="Saludos desde PrjLambdaWiki, Lambda Simulate..."
        )
    except Exception as e:
        return dict(
            statusCode=500,
            body=str(e)
        )
def s3hook(event, context):
    path = Path("/tmp") / event["Records"][0]["s3"]["object"]["eTag"]
    with path.open("w") as f:
        f.write(json.dumps(event))

    print(json.dumps(OrderedDict([
        ("statusCode", 200),
        ("body", "result")
    ])))

    return 0
```

### Soporte AWS CloudFront Edge Lambda:
```sh
$ npm install --save-dev serverless-offline-edge-lambda
$ serverless plugin install -n serverless-offline-edge-lambda
```
Ejemplo de Configuración
- **serverless.yml**
```yml
(pendinte)
```
- **handler.yml**
```yml
(pendiente)
```

### Soporte Acceso a MySQL desde AWS Python Lambda:
```sh
$ pip install pymysql
$ vi mysql_config.py
# Archivo local para configurar Mysql local o RDS
db_username = "usuario"
db_password = "clave"
db_name = "Nombre_Schema"
db_endpoint = "ip" o "dns" 
db_port ="3306"
```

```py
import sys
import logging
import mysql_config
import pymysql

rds_host  = mysql_config.db_endpoint
name = mysql_config.db_username
password = mysql_config.db_password
db_name = mysql_config.db_name
db_port = mysql_config.db_port

logger = logging.getLogger()
logger.setLevel(logging.INFO)

try:
    conn = pymysql.connect(rds_host, user=name, port=db_port,
                           passwd=password, db=db_name, connect_timeout=5)
except:
    logger.error("ERROR: Unexpected error: Could not connect to MySql instance.")
    sys.exit()

logger.info("SUCCESS: Connection to RDS mysql instance succeeded")
```

### Instalando Cloud9 Localmente:
Crear un directorio para la instalación de Cloud9 Localmente:
```sh
$ mkdir cloud9
$ git clone https://github.com/c9/core.git c9sdk
$ cd c9sdk
$ scripts/install-sdk.sh
```
Ejecutando Cloud9 Localmente:
```sh
$ node server.js
```
Accediendo a CLoud9 Localmente:
```sh
$ firefox  http://localhost:8181/ide.html
```
Si deseas iniciar en tu direcorio de proyecto el IDE de Cloud9, debes iniciar el servidor node con el siguiente parametro:
-w: Diretorio de trabajo.
```sh
$ node server.js -w /<miruta>/<miproyecto>
$ firefox  http://localhost:8181/ide.html
```

### Instalando DynamoDB Localmente con Docker-Compose:
Crear un directorio para el Contenedor Docker de DynamoDB.

```sh
$ mkdir -p dck_Dynamobd/db_data
$ cd dck_Dynamobd
$ vi docker-compose.yml
```
- **docker-compose.yml**
```txt
version: '3'
services:
  db_dynamodb:
    container_name: db_dynamodb
    image: amazon/dynamodb-local:latest
    ports:
      - "8000:8000"
    volumes:
      - "/dockerImg/10.Docker/dynamodb/db_data:/data"
```
Crear el Contenedor Docker.
```sh
$ docker-composer up -d
```
Accediendo a DynamoDB Localmente con el AWS CLI:
* Listar las Tablas Existentes
```sh
$ aws dynamodb list-tables --endpoint-url http://localhost:8000
```
* Estructura de una  Tablas Existentes:
```sh
$ aws dynamodb describe-table --table-name <TablaName> --endpoint-url http://localhost:8000
```
* Estado de una  Tablas Existentes:
```sh
$ aws dynamodb describe-table --table-name $1 --endpoint-url http://localhost:8000 | grep TableStatus
```
* Crear una Tabla DynamoDB:
```sh
$ aws dynamodb create-table --table-name cliente \
  --attribute-definitions \
        AttributeName=Id,AttributeType=N \
        AttributeName=Rut,AttributeType=S \
        AttributeName=Nombre,AttributeType=S \
        AttributeName=Apellido,AttributeType=S \
  --key-schema \
        AttributeName=Id,KeyType=HASH \
        AttributeName=Id,KeyType=RANGE \
  --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=5 \
  --endpoint-url http://localhost:8000
```


## SACACIngeniería
### SACACI Chile


