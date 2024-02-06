# DevOps TP1


# Postgresql Setup

## Build
`docker network create app-network`

`docker build -t ycathelain/db`

## Run
`docker run -d --net=app-network --name=db ycathelain/db`

`docker run -p "8090:8080" --net=app-network --name=adminer -d adminer`

## Mount volume

`docker run -d -v sql:/var/lib/postgresql/data --net=app-network --name=db ycathelain/db`

## Dockerfile

```bash
FROM postgres:14.1-alpine

ENV POSTGRES_DB=aled \
   POSTGRES_USER=aled \
   POSTGRES_PASSWORD=aled

COPY ./createSchema.sql /docker-entrypoint-initdb.d/
COPY ./insertData.sql /docker-entrypoint-initdb.d/
```

# Springboot App

## Build

`docker build -t ycathelain/simpleapi`

## Run

`docker run -p "8080:8080" --name simpleapi --net app-network ycathelain/simpleapi`

## Dockerfile

```bash
# Build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
```

# Front-end Server

## Build

`docker build -t ycathelain/http`

## Run 

`docker run -dit -p "8000:80" --net app-network --name http ycathelain/http`

## Dockerfile

```bash

FROM httpd:2.4
COPY ./ /usr/local/apache2/htdocs/
COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf

```



# Question

## Why do we need a multistage build? And explain each step of this dockerfile.

Un multistage build est utilisé pour réduire la taille de l'image final qui va être généré sur docker. Dans l'image final, dans notre cas avec Maven, elle contiendra seulement le JAR du projet et le JRE au lieu d'embarqué le JDK entier.
Cela facilite aussi le build de l'image étant donné qu'elle est plus petite en terme de stockage car moins d'installation lié au build dessus.


## Dockerfile Springboot explication : 

### Build 

`FROM maven:3.8.6-amazoncorretto-17 AS myapp-build` Cette ligne va chercher l'image pour l'environnement Java

`ENV MYAPP_HOME /opt/myapp` Cette ligne stock le chemin dans une variable d'environnement

`WORKDIR $MYAPP_HOME` Indique le répertoire de travail 

`COPY pom.xml .` Copie le fichier de config Maven pom.xml dans le répertoire de travail

`COPY src ./src` Copie le src du projet Java dans un répertoire src du répertoire de travail

`RUN mvn package -DskipTests` Installe toutes les dépendances Maven en skippant les tests (processus long)


### Run

`FROM amazoncorretto:17` Configure l'image de base pour notre image

`ENV MYAPP_HOME /opt/myapp` Stock le chemin /opt/myapp dans la variable d'environnement MYAPP_HOME

`WORKDIR $MYAPP_HOME` Configure le répertoire de travail selon le contenu de la variable d'environnement

`COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar` Copie le JAR généré dans l'image Docker

`ENTRYPOINT java -jar myapp.jar` Configure la commande qui va être run par défaut dans le conteneur au lancement


## Docker Compose

`docker compose up` Build, recrée et lance les conteneurs, peut être lancé avec ou sans l'option -d qui permet de run en background, ou si non présent permet de voir le lancement des conteneurs.

`docker compose down` Stop et supprime tous les conteneurs, images, volumes crée par le docker compose.

`docker compose build` Build ou rebuild tous les services du docker compose

`docker compose logs` Permet d'accéder aux logs des services du docker compose

`docker compose ps` Liste tous les conteneurs en cours d'exécution en relation avec la configuration docker-compose.yaml

`docker compose restart` Restart tous les services du docker compose.

`docker compose stop/start` Stop ou start les services.

## Docker-compose.yaml

```yaml

version: '3.7'

services:
    backend:
        build: './springboot-db/simple-api-student'
        networks:
            - my-network
        environment:  
            - SPRING_DATASOURCE_URL=jdbc:postgresql://database/aled
            - SPRING_DATASOURCE_USERNAME=aled
            - SPRING_DATASOURCE_PASSWORD=aled

    database:
        build: './TP1'
        networks:
        - my-network

    httpd:
        build: './http'
        ports:
        - "80:80"
        networks:
        - my-network
        depends_on:
        - backend

networks:
    my-network:

```

Ce docker-compose configure une application 3-tier composé de trois conteneurs. 

   1. Un conteneur database qui va monté une base de données postgres pour nos applications
   2. un conteneur backend qui monte une api Springboot pour exposé des endpoints
   3. un conteneur httpd qui monte un server Apache front-end avec un reverse proxy pour redirigé vers les endpoints Springboot.

Le network my-network permet à tous les conteneurs de fonctionner ensemble dans un même network.

## Docker publish 

`docker login` Permet de se connecter à DockerHub et aux répository.

`docker tag devops-tp1-database ycathelain/devops-tp1-database` tag une image Docker pour qu'elle puisse être publiée sur un répository.

`docker push ycathelain/devops-tp1-database` Créé un répository et push l'image docker sur ce répository.

On peut faire la même chose pour les images `ycathelain/devops-tp1-backend` et `ycathelain/devops-tp1-httpd`