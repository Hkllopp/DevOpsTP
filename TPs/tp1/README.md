# DevOps TP1

## Group
Loïs Motel
Augustin Bar

## Compte rendu

Création de la hiérarchie de fichier suivante :

TP1
----database
--------Dockerfile
----backendApi
--------Dockerfile
----httpserver
--------Dockerfile

### Création de la Database

First, we'll setup a minimal Docker file for the database, with env variables for the DB name, the user login and the password:

```dockerfile
FROM postgres:11.6-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd
```

There is a good practice to remember when using the FROM in dockerfiles. It is to alway reference the version of the image we are using. That way, you avoid incompatibility caused by an upgraded version when you'll later try to reuse your dockerfile.

To build this, well use the "docker build" command without forgeting to set a tag for our new docker image:

```bash
sudo docker build -t sirlewis/database .
```

We'll also use admin, available on docker hub, as a lightweight SQL client to test our database. It's easy to setup, either with a link like this (this methode is deprecated): 
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

I think you'll notice the -v option on the second option. This one allow the database data to persist across container start and stop cycle. This is mandatory to keep your database consistant over time!

There is another good thing to know. You can use a "docker-entrypoint-initdb.d" folder with your docker file to allow it when building to play SQL script to create the database structure and to insert data into it. I'll play all script by alphabetical order. 

You'll also need to modify a little the Dockerfile, like this: 

```dockerfile
FROM postgres:11.6-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd

COPY * ./docker-entrypoint-initdb.d/
```

After a last docker build and running your new image into a container, we'll have a fully working database. 
### Backend API

Create your java app  "Main.java":

```java
public class Main{

	public static void main(String[] args){
		System.out.println("Hello World!");
	}

}
```

Create a Dockerfile referencing the "openjdk" image:

```dockerfile
#Get lastest openjdk image
FROM openjdk
#Copy the Main.java file to the build container
COPY Main.java .
#Build the java app from the build container
RUN javac Main.java
#Start the java app from the run container
CMD ["java", "Main"]
```

Build the docker image:

```bash
sudo docker build -t sirlewis/hello-world .
```

Then run a new container using this fresh builded image:

```bash
sudo docker run sirlewis/hello-world
```

You'll see this message in you shell:
Hello World!

Now with the multistep build:

```dockerfile
FROM openjdk:11 as builder

COPY Main.java /usr/src/

RUN javac /usr/src/Main.java

FROM openjdk:11-jre

COPY --from=builder /usr/src/Main.class .

CMD ["java", "Main"]
```

Same result, with a shitload of download <3

### HTTP Server

sudo docker run -d --name web-server --network neverland -p 80:80 sirlewis/web-server

sudo docker run --network neverland --name simple-api -p 8080:8080 sirlewis/maven-api