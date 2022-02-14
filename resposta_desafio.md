# QUESTÃO 1

Para que haja persistência de dados ao criamos containers, utiliza-se os volumes do docker. Assim, antes de rodar os containers, criamos os volumes com o seguinte comando:

`docker volume create [NOME_DO_VOLUME]`

Assim, criaremos 4 volumes, sendo um para cada container: MongoDB, MariaDB, PostgreSQL e Redis.

```
docker volume create mongodb_vol
docker volume create mariadb_vol
docker volume create postgres_vol
docker volume create redis_vol
```

Agora, podemos rodar os comandos para criação dos containers seguindo o seguinte padrão:

```
docker container run --name [NOME] -d -v [DOCKER_VOLUME]:[PATH_NO_CONTAINER] -e [USUARIO] -e [SENHA] -p [PORTA_HOST]:[PORTA_CONTAINER] [IMAGEM_DO_DB]:[TAG]
```
## MongoDB:
`docker container run --name mongodb -d -v mongodb_vol:/data/db -e MONGO_INITDB_ROOT_USERNAME=mongouser -e MONGO_INITDB_ROOT_PASSWORD=mongopwd -p 27017:27017 mongo:4.4.3`

## MariaDB
`docker container run --name mariadb -d -v mariadb_vol:/var/lib/mysql -e MARIADB_ROOT_HOST=password -p 3306:3306 mariadb:10.6.4`

## PostgreSQL
`docker container run --name postgres -d -v postgres_vol:/var/lib/postgresql/data -e POSTGRES_PASSWORD=password -p 5432:5432 postgres:9.6.23-alpine3.14`

## Redis
`docker container run --name redis -d -v redis_vol:/data -p 6379:6379 redis:6.2.6 redis-server --appendonly yes`

# QUESTÃO 2

Executando ferramentas administrativas com interface web dos bancos da questão anterior via container. As interfaces são acessadas via `localhost:[PORTA_DO_HOST]`.

## MongoDB (Mongo Express)
Criando uma network para a comunicação entre os containers:
`docker network create mongo_network`

Subindo o container do MongoDB:
`docker container run --name mongodb --network mongo_network -d -v mongodb_vol:/data/db -e -e MONGO_INITDB_ROOT_USERNAME=mongouser -e MONGO_INITDB_ROOT_PASSWORD=mongopwd -p 27017:27017 mongo:4.4.3`

Subindo o container do Mongo Express:
`docker container run --name mongoexpress --rm --network mongo_network -e ME_CONFIG_MONGODB_URL=mongodb://mongouser:mongopwd@mongodb:27017 -p 8081:8081 mongo-express:1.0.0-alpha.4`

## MariaDB (phpmyadmin)
Criando uma network para a comunicação entre os containers:
`docker network create maria_network`

Subindo o container MariaDB:
`docker container run --name mariadb -d --network maria_network -v mariadb_vol:/var/lib/mysql -e MARIADB_ROOT_PASSWORD=password -p 3306:3306 mariadb:10.6.4`

Subindo o container PHPMyAdmin:
`docker container run --rm --name phpmyadmin -d --network maria_network --link mariadb:db -p 8080:80 phpmyadmin:5.1.1`

## PostgreSQL (pgAdmin)
Criando uma network para a comunicação entre os containers:
`docker network create postgres_network`

Subindo o container PostgreSQL:
`docker container run --name postgres -d --network postgres_network -v postgres_vol:/var/lib/postgresql/data -e POSTGRES_PASSWORD=password -p 5432:5432 postgres:9.6.23-alpine3.14`

Subindo o container pgAdmin:
`docker container run --rm --name pgadmin -d --network postgres_network -e "PGADMIN_DEFAULT_EMAIL=user@email.com" -e "PGADMIN_DEFAULT_PASSWORD=password" -e "PGADMIN_LISTEN_PORT=5051" -p 5051:5051 dpage/pgadmin4:6`

## Redis (redis-commander)
Criando uma network para a comunicação entre os containers:
`docker network create redis_network`

Subindo o container Redis:
`docker container run --name redis -d --network redis_network -v redis_vol:/data -p 6379:6379 redis:6.2.6 redis-server --appendonly yes`

Subindo o container Redis-Commander:
`docker container run --rm --name redis-commander -d  --network redis_network -e REDIS_HOSTS=local:redis:6379 -p 8081:8081 rediscommander/redis-commander`

# QUESTÃO 3

Repositório da aplicação Node.js: https://github.com/raphaelmb/conversao-temperatura

Repositório da aplicação Python: https://github.com/raphaelmb/conversao-distancia

Repositório da aplicação .NET: https://github.com/raphaelmb/conversao-peso 

# QUESTÃO 4

Foi adicionado um arquivo docker-compose.yaml ao projeto para subir a aplicação e banco de dados com um único comando:

```
version: "3.8"

networks:
  rotten-potatoes:
    driver: bridge

volumes:
  mongo_rotten_potatoes_volume:

services:
  app:
    build: .
    ports:
      - 5000:5000
    depends_on:
      - db
    networks:
      - rotten-potatoes
    environment:
      MONGODB_DB: admin
      MONGODB_HOST: db
      MONGODB_PORT: 27017
      MONGODB_USERNAME: mongouser
      MONGODB_PASSWORD: mongopwd

  db:
    image: mongo:5.0.5
    restart: always
    networks:
      - rotten-potatoes
    ports:
      - 27017:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: mongouser
      MONGO_INITDB_ROOT_PASSWORD: mongopwd
    volumes:
      - mongo_rotten_potatoes_volume:/data/db
```

Segue o link para o repositório: https://github.com/raphaelmb/rotten-potatoes

# QUESTÃO 5



