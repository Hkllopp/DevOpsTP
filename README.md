# DevOpsTP

## Commandes utiles

- Pour build une image sur DockerHub à partir d'un Dockerfile `docker build -t <YOUR_USERNAME_IN_DOCKER>/<NOM_DE_LAPP> <DIRECTORY_TO_DOCKERFILE>`

- Pour lancer une image docker à partir de dockerhub `docker run -d -p <PORT_ACCESS:PORT_LISTEN> --name <NOM_DE_LIMAGE> <YOUR_USERNAME_IN_DOCKER>/<NOM_DE_LAPP>`. L'option -d (detached) permet de ne pas voir les outputs (logs) du container

- Pour connaitre les processus docker en cours `docker ps` et l'intégralité des processus docker `docker ps -a`

- Pour arrêter puis tuer un processus docker `docker stop <PROC_ID_OR_NAME>` puis `docker rm <PROC_ID_OR_NAME>`. On peut concatener ces commandes en faisant `docker rm -f <PROC_ID_OR_NAME>`

- Pour stoper et supprimer tous les processus docker `docker stop $(docker ps -aq)&&docker rm $(docker ps -aq)`

- Pour connaitre tous les programmes qui écoutentn à certains ports : `lsof -nP | grep LISTEN`

- Pour arrêter un service `service <NOM_OU_ID> stop`

## Database

- Pour utiliser adminer afin de voir visualiser une db dans son navigateur :
    1. `docker run --name <NOM_IMAGE> -d -p <POST_ACCESS:PORT_LISTEN> --name <NOM_IMAGE> --link <NOM_IMAGE_ASSOCIEE> adminer`
    2. Accéder à `localhost:8080` dans le navigateur

Ex :
`docker build -t baraugustin/tp01 .`
`docker run --name tp01 -p 5432:5432 baraugustin/tp01` *On a pas mis le -d car on veut que les logs s'affichent dans la console*
`docker run --name clientAdminer -p 80:8080 --link tp01 adminer`
On se connecte ensuite à la bd via `localhost` et on rentre les informations suivantes pour se connecter
| System   | PostgreSQL |
|----------|------------|
| Server   | tp01       |
| Username | usr        |
| Password | pwd        |
| Database | db         |

line command : `clear&&docker stop $(docker ps -aq)&&docker rm $(docker ps -aq)&&docker build -t baraugustin/tp01 . && docker run --name tp01 -v /my/own/datadir:/var/lib/postgresql/data -d  -p 5432:5432 baraugustin/tp01 && docker run --name clientAdminer -d -p 80:8080 --link tp01 adminer`

## Java

Lorsqu'on créé un fichier en java (.java), on peut le compiler avec la commande `javac <FICHIER.java>` afin de créer l'environnement qui sera utilisé lors de l'execution. Pour lancer le code, on fait `java <NOM_DE_LA_CLASSE_MAIN_SANS_EXTENSIONS>`

Du coup, pour créer un Dockerfile qui va build puis executer un proogramme java, on écrit :

```FROM openjdk:11
# Build Main.java
CMD ["javac","Main.java"]
COPY Main.class /usr/src/

FROM openjdk:11-jre
# Copy ressource from previous stage
COPY --from=0 /usr/src/Main.class ./
# Run java code with the JRE
CMD ["java","Main"]
```

## Networking

Pour connecter plusieurs containers entre eux, on peut utiliser les links ou les networks. Les links étant dépréciés, on préferera utiliser les networks.
On commence par créer un network docker avec `docker network create <NETWORK_NAME>`.
On peut ensuite lancer les containers en spécifiant bien le network auquel il est connecté : `docker run -d --name <CONTAINER_NAME> --network <NETWORK_NAME> <CONTAINER>`
Pour inspecter le réseau docker, on peut faire la commande `docker network inspect bridge` et pour inspecter un réseau en particulier `docker network inspect <NETWORK_NAME>`

Par exemple, on a fait :

```docker network create myFirstNetwork
docker run --name running-database -p 5432:5432 --network myFirstNetwork baraugustin/tp01/database
docker run --name databaseAPI -p 8080:8080 --network myFirstNetwork baraugustin/tp01/backendapi2/simpleapi
```

On a créé un network myFirstNetwork et on a utilisé le nom running-database pour des soucis de compatibilité

## Reverse Proxy via Apache Server

On utilise un serveur Apache pour faire un reverse proxy : la connexion à l'API va d'abord passer par le serveur. Cette configuration a beaucoup d'avantages au niveau sécurité, optimisation de charge, etc...
On va ouvrir l'API sur le port :8080 et lorsque le reverse proxy recevra une ip en :80 (HTTP par défaut), il le redirigera vers l'API.

Exemple :

```docker build -t baraugustin/tp01/apacheserver .
docker run  --name apacheserver --network myFirstNetwork -p 80:80 baraugustin/tp01/apacheserver

```

On peut accéder au serveur via `localhost` via le navigateur.
On peut exécuter des commandes dans un container avec `docker exec <CONATINER_NAME> <COMMAND>`

## Docker Compose

Docker compose permet d'automatiser le lancement des containers. Pour cela, on précise le chemin de chaque Dockefile ainsi que les options de run

## Debugging

Si jamais un container ne veut pas se lancer pour un problème de port, on peut inspecter les ports avec `netstat -ltnp | grep -w '<PORT>'`. On peut ensuite tuer (SIGKILL) le processus avec `kill -9 <PROC_ID>`.
