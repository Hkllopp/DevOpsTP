# DevOpsTP

## Comamndes utiles

- Pour build une image sur DockerHub à partir d'un Dockerfile `docker build -t <YOUR_USERNAME_IN_DOCKER>/<NOM_DE_LAPP> <DIRECTORY_TO_DOCKERFILE>`

- Pour lancer une image docker à partir de dockerhub `docker run -d -p <PORT_ACCESS:PORT_LISTEN> --name <NOM_DE_LIMAGE> <YOUR_USERNAME_IN_DOCKER>/<NOM_DE_LAPP>`

- Pour connaitre les processus docker en cours `docker ps` et l'intégralité des processus docker `docker ps -a`

- Pour arrêter puis tuer un processus docker `docker stop <PROC_ID_OU_NAME>` puis `docker rm <PROC_ID_OU_NAME>`

- Pour stoper et supprimer tous les processus docker `docker stop $(docker ps -aq)&&docker rm $(docker ps -aq)`

- Pour connaitre tous les programmes qui écoutentn à certains ports : `lsof -nP | grep LISTEN`

- Pour arrêter un service `service <NOM_OU_ID> stop`

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

1-line command : `clear&&docker stop $(docker ps -aq)&&docker rm $(docker ps -aq)&&docker build -t baraugustin/tp01 . && docker run --name tp01 -v /my/own/datadir:/var/lib/postgresql/data -d  -p 5432:5432 baraugustin/tp01 && docker run --name clientAdminer -d -p 80:8080 --link tp01 adminer`

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
