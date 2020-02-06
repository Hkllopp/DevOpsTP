# Report DevOps Lessons

## Group
 - Loïs Motel
 - Augustin Bar

# TP1: Docker & Maven

## Commandes utiles

- Pour build une image sur DockerHub à partir d'un Dockerfile `docker build -t <YOUR_USERNAME_IN_DOCKER>/<NOM_DE_LAPP> <DIRECTORY_TO_DOCKERFILE>`

- Pour lancer une image docker à partir de dockerhub `docker run -d -p <PORT_ACCESS:PORT_LISTEN> --name <NOM_DE_LIMAGE> <YOUR_USERNAME_IN_DOCKER>/<NOM_DE_LAPP>`. L'option -d (detached) permet de ne pas voir les outputs (logs) du container

- Pour connaitre les processus docker en cours `docker ps` et l'intégralité des processus docker `docker ps -a`

- Pour arrêter puis tuer un processus docker `docker stop <PROC_ID_OR_NAME>` puis `docker rm <PROC_ID_OR_NAME>`. On peut concatener ces commandes en faisant `docker rm -f <PROC_ID_OR_NAME>`

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

line command : `clear&&docker stop $(docker ps -aq)&&docker rm $(docker ps -aq)&&docker build -t baraugustin/tp01 . && docker run --name tp01 -v /my/own/datadir:/var/lib/postgresql/data -d  -p 5432:5432 baraugustin/tp01 && docker run --name clientAdminer -d -p 80:8080 --link tp01 adminer`

## Database 

First, we'll setup a minimal Docker file for the database, with env variables for the DB name, the user login and the password:

```dockerfile
FROM postgres:11.6-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd
```

There is a good practice to remember when using the FROM in dockerfiles. It is to always reference the version of the image we are using. That way, you avoid incompatibility caused by an upgraded version when you'll later try to reuse your dockerfile.

To build this, well use the "docker build" command without forgetting to set a tag for our new docker image:

```bash
sudo docker build -t sirlewis/database .
```

We'll also use admin, available on docker hub, as a lightweight SQL client to test our database. It's easy to setup, either with a link like this (this method is deprecated): 
```bash
sudo docker run -d --name database -p 8888:5432 sirlewis/database
sudo docker run -d --link database:db -p 8080:8080 adminer
```
(you'll use the name db as the server name in adminer)

or with a network like this: 

```bash 
sudo docker run -d --name database --network neverland -p 8888:5432 -v /home/loism/Documents/ sirlewis/database
sudo docker run -d --name adminer --network neverland -p 8080:8080 adminer
```
(this time, you'll use database as server name)

I think you'll notice the -v option on the second option. This one allows the database data to persist across the container start and stop cycles. This is mandatory to keep your database consistent over time!

There is another good thing to know. You can use a "docker-entrypoint-initdb.d" folder with your docker file to allow it when building to play SQL script to create the database structure and to insert data into it. I'll play all scripts in alphabetical order. 

You'll also need to modify a little the Dockerfile, like this: 

```dockerfile
FROM postgres:11.6-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd

COPY * ./docker-entrypoint-initdb.d/
```

After a last docker builds and running your new image into a container, we'll have a fully working database. 

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

# TP2: Travis CI & SonarCloud 

## Setup Travis Acount

Create a Github account or log into your existing one.

Fork sample-application-students from the takima github project.

Create a Travis account (SSO with github account) and activate it from the profile page. You'll then be able to select which project you want to link to travis.

## Setup Project with travis 

Now, you'll have to setup a Travis yaml file `.travis.yml` into your project to allow Travis to interact with it when you push it to git. 

My project structure is like that: 

```
sample-application-students
----.git
----.gitignore
----.travis.yml
----README.md
----sample-application-db-changelog-job
----sample-application-http-api-server
----...
```

You can see now where to put the `.travis.yml` file, at the root of your project. 

To make it work with our project that use maven, we'll use two simple lines: 

```yaml
language: java
script: 
  - mvn clean verify
```
that's the basic content of our file. This command will execute maven on Travis whit a special option allowing it to clear all previous build files to avoid incompatibilities with the new code added. So, it'll build everything from scratch.

Now that it is all setup, we just need to push it our git repository and wait for Travis to do the job. 

As explained in the TP, Travis will use two things from our project to help running test on our http api server. First, there is the db-changelog-job that will help create schemes for the test database. This database will be a docker container mounted thank's to a dependency specified inside our pom.xml for the maven execution. As travis will run the content of our project, it will mount the container and use our job to populate it with test content. 

Now that's it. If everything is okay, on the Travis interface, we can see the last build number with the green message "passed". 

## Deploy to docker hub

Now that we have a CI for our project, we'll need to add a CD (continuous delivery) for our build to be deployed, here on our docker hub repository.

First step, creating a new branch of our git project. The easiest way is a git checkout with the branch option: `git checkout -b dev`


With this, we are directly switch to our dev branch after its creation.
Don't forget to add your branch to the remote, but git will show a warning in case of a push on this branch. 

After that, we have to allow Travis to connect to our dockerhub when the build is deployed. To do this, we'll use two things: 
  - Secure environment variable from Travis, to hide our credentials
  - Connection token from docker hub, to avoid spreading our credentials everywhere

For the first one, we just have to go to our pipeline options and add two variables, one for the user and one for the tocken: `$user $token`
We'll add them later to our .travis.yml when we'll push our code. 

Now what we need to do is to create one Dockerfile for each app of our project, to allow Travis to build them after all the tests and to push them to our docker hug repository. 
We do it like this: 

```dockerfile
FROM openjdk:11-jre
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY ./target/*-exec.jar ./myapp.jar

ENTRYPOINT java -jar myapp.jar

```
The two DockerFiles are the same, because here we just need to specify how to run apps, because Travis already built the sources. 

Then, we will tell Travis to build these docker images and push them to docker hub: 

```yaml
language: java
script:
  - mvn clean verify 
  - docker build -t $user/backend sample-application-http-api-server/.
  - docker build -t $user/database sample-application-db-changelog-job/.
  - docker login -u $user -p $token
  - docker push $user/backend
  - docker push $user/database
```

You can now see why we've used environment variable. First, to avoid the need to change everything by hand if you change your credentials, but also to avoid displaying them to the world on your public github repository!

Last steps for this part: 
  - creating a build pipeline for the dev branch on Travis
  - creating a repository for each app on docker hub with the same name as your new docker images, or they won't be pushed 

And that's it, now you just have to wait for the build to finish and the deployment to push everything on your docker hub repositories. 

## Setup Quality Gate 

Now, last but not least, the quality gate. 
We used Sonarcloud, the cloud version of Sonarqube to analyse our code and generate quality marks for our project for things like test code coverage, technical debt, code smell (bad practices in code) etc...

First thing, creating an account. Like Travis, we can use our github account to create a Sonarcloud one. that done, we have to first create a project by selecting our github repository.

When it's done, we'll have to tell Travis to use Sonarcloud to analyse the code when running maven. For that purpose, we first need to add another environment variable to our Travis CI: `$soncloudtoken`

This token will be used to allow our Travis to connect to our Sonarcloud project and tell him to analyse our code. In return, Sonarcloud will generate his report about our last build.

Finally, the last step is to configure our `.travis.yml` by adding an add-on to it, with our keys from Sonarcloud (organization key, project key): 

```yml
language: java
addons:
  sonarcloud:
    organization: "sirlewis" # the key of the org you chose at step #3
    token:
      secure: $soncloudtoken # encrypted value of your token
script:
  - mvn clean verify sonar:sonar -Pcoverage -Dsonar.projectKey=SirLewis_sample-application-students
  - docker build -t $user/backend sample-application-http-api-server/.
  - docker build -t $user/database sample-application-db-changelog-job/.
  - docker login -u $user -p $token
  - docker push $user/backend
  - docker push $user/database
```

You can see here that we added some options to `mvn clean verify`. These options tell Travis to use sonarcloud with maven and what project to target for the analysis. 

And now, we can push the code, which will run a new buikd, play tests and generate a report for the code quality on sonarcloud. Don't forget to check on what branch your Sonarcloud report is done ;)