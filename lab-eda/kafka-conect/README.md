# Lab Eda - Kafka Connect


## Disclaimer
> **As configurações dos Laboratórios é puramente para fins de desenvolvimento local e estudos**


## Pré-requisitos?
* Docker
* Docker-Compose


# Instalação Kafka 

[LAB EDA](lab-eda//README.md)


## Kafka Connect

![Exemplo Kafka Conect](../../content/kafka-connect-minio.png)


### Realizando download do plugins Debezium para PostGres (Source) 

```
cd lab-eda/kafka-conect

mkdir plugin

curl -sfSL https://repo1.maven.org/maven2/io/debezium/debezium-connector-postgres/2.1.3.Final/debezium-connector-postgres-2.1.3.Final-plugin.tar.gz | tar xz -C plugin

```

Criando a imagem junto com o plugin do Debezium Postgres


```
 docker image build -t <<usuario>>/kafka-connet-debezium-lab-v0  -f Dockerfile .
 
```

Vamos enviar a imagem para o dockerhub ??
https://hub.docker.com/

```
docker image push <<conta>>/kafka-connet-debezium-lab-v0
```

> As imagens customizadas encontra-se no https://hub.docker.com/


Container  criado? ...mas antes

Altere o arquivo ambiente/docker-compose.yaml da imagem criada no serviço `connect`


No diretório `/lab-eda/ambiente` execute o comando abaixo

```
cd ../ambiente/

docker-compose up -d  connect

docker container ls
```

Listando os plugins existentes, os padrões da imagem e do debezium que foi inserido na imagem, via arquivo `Dockerfile`

```
docker exec -it kafkaConect curl  http://localhost:8083/connector-plugins
```

## Configurando o Conector Postgres


### Provisionando Banco de dados Postgres e a ferramenta PgAdmin

```
docker-compose up -d postgres pgadmin
```

Acesso para o PgAdmin http://localhost:5433/


* Login: lab-pgadmin4@pgadmin.org
* Senha : postgres    

* Nome do server: postgres
* Nome do Host Name: postgres
* database: postgres
* Username: postgres
* password: postgres

### Tela de login do PgAdmin
![Exemplo Kafka Conect](../../content/login-pgadmin.png)


### Inserindo um server
![Exemplo Kafka Conect](../../content/add-server.png)

### Configurando o server
![Exemplo Kafka Conect](../../content/conect-pgadmin.png)

### ...Se tudo deu certo o banco de exemplo com suas tabelas
![Exemplo Kafka Conect](../../content/tabelas.png)

### Criando os Conectores

*API rest do kafka Connect*
https://docs.confluent.io/platform/current/connect/references/restapi.html


Criando o conector PostGres

```

cd ../kafka-conect

http PUT http://localhost:8083/connectors/connector-postgres/config < conector-postgres.json

```


* Observando o arquivo `conector-postgres.json` 

Algumas informações básicas sobre o connector:


* `spec.class`: Nome da classe do conector que está dentro do plugin debezium
* `spec.config.database.hostname` e `spec.config.database.port`: endereço IP ou nome de host para sua instância Sql Server, bem como a porta (por exemplo 1433)
* `spec.config.database.user` e `spec.config.database.password`: nome de usuário e senha para sua instância Sql Server
* `spec.config.database.dbname`: nome do banco de dados
* `spec.config.database.server.name`: Nome lógico que identifica e fornece um namespace para o servidor / cluster de banco de dados Sql Server específico que está sendo monitorado.
* `spec.config.table.whitelist`: lista separada por vírgulas de regex especificando quais tabelas você deseja monitorar para a captura de dados alterados


Listando os conectores

```
http http://localhost:8083/connectors/
```

Verificando o status dos conectores

```
http http://localhost:8083/connectors/connector-postgres/status

```

### E o Akhq ?


### Testando o Conector

Vamos inserir alguns registros nas tabelas e listar os tópicos do Kafka



### Inserir um registro na tabela `inventory.products`


![Exemplo Kafka Conect](../../content/insert.png)

```
INSERT INTO inventory.products(	id, name, description, weight)
VALUES (110, 'Lapis', 'O melhor', 1);
```

Listando os tópicos


```
docker exec -it kafka-broker /bin/bash
kafka-topics --bootstrap-server localhost:9092 --list 
```



*Consumindo mensagem postgres.inventory.products Datasource Postgres*

```
kafka-console-consumer --bootstrap-server localhost:9092 --topic postgres.inventory.products --from-beginning
```


#### Api Rest Kafka Connect


```
exit

http http://localhost:8083/connectors/connector-postgres/status

```

Interagindo com os connetores

```
http PUT http://localhost:8083/connectors/connector-postgres/pause
http http://localhost:8083/connectors/connector-postgres/status
http PUT http://localhost:8083/connectors/connector-postgres/resume
```

### Configurando conector SYNC do MinIO

Subindo o serviço do MinIO

```
cd ../ambiente
docker-compose up -d minio
```

> http://localhost:9001/login


Intalando o plugin do MinIO


```
wget https://api.hub.confluent.io/api/plugins/confluentinc/kafka-connect-s3/versions/10.5.5/archive

unzip archive

mkdir -p plugin/kafka-connect-s3

mv confluentinc-kafka-connect-s3-10.5.5/lib/* plugin/kafka-connect-s3/

docker image build -t fernandos/kafka-connet-debezium-lab-v1  -f Dockerfile .
```

Mudando a imagem do Kafka Conect

```
cd ../ambiente/

docker-compose up -d  connect
```

Instalando o conector do MinIO

```
cd ../kafka-conect

http PUT http://localhost:8083/connectors/connector-minio/config < conector-minio.json
```


## Desafio



*Fazer o Sinc para o Mongodb*

![Exemplo Kafka Conect](../../content/sinc.png)


>Dicas

```
//Download do Plugin do Kafka Connect
 wget https://repo1.maven.org/maven2/org/mongodb/kafka/mongo-kafka-connect/1.6.1/mongo-kafka-connect-1.6.1-all.jar -P plugin

//Instalação do Mongodb mongo-connect
docker-compose up -d mongo-connect

 http PUT http://localhost:8083/connectors/sinc-mongodb/config < sinc-mongodb.json

 ```

 > Lab Mongodb
 [LAB NOSQL](../../lab-nosql/README.md)

 https://github.com/nandorsilva/arc-dados/tree/main/lab-nosql
