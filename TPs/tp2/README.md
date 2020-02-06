# TP2: Travis CI & SonarCloud 

## Setup Travis Acount

Create a Github account or log into your existing one.

Fork sample-application-students from the takima github project.

Create a Travis account (SSO with github account) and activate it from the profile page. You'll then be able to select which project you want to link to travis.

## Setup Project with travis 

Now, you'll have to setup a Travis yaml file (.travis.yml) into your project to allow Travis to interact with it when you push it to git. 

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

You can see now where to put the .travis.yml file, at the root of your project. 

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

First step, creating a new branch of our git project. The easiest way is a git checkout with the branch option: 

```git
git checkout -b dev
```

With this, we are directly switch to our dev branch after its creation.
Don't forget to add your branch to the remote, but git will show a warning in case of a push on this branch. 

After that, we have to allow Travis to connect to our dockerhub when the build is deployed. To do this, we'll use two things: 
  - Secure environment variable from Travis, to hide our credentials
  - Connection token from docker hub, to avoid spreading our credentials everywhere

For the first one, we just have to go to our pipeline options and add two variables, one for the user and one for the tocken: $user $token
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

When it's done, we'll have to tell Travis to use Sonarcloud to analyse the code when running maven. For that purpose, we first need to add another environment variable to our Travis CI: $soncloudtoken

This token will be used to allow our Travis to connect to our Sonarcloud project and tell him to analyse our code. In return, Sonarcloud will generate his report about our last build.

Finally, the last step is to configure our .travis.yml by adding an add-on to it, with our keys from Sonarcloud (organization key, project key): 

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

You can see here that we added some options to the "mvn clean verify" command. These options tell Travis to use sonarcloud with maven and what project to target for the analysis. 

And now, we can push the code, which will run a new buikd, play tests and generate a report for the code quality on sonarcloud. Don't forget to check on what branch your Sonarcloud report is done ;)