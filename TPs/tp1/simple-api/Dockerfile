#FROM openjdk:11 as builder
#COPY Main.java /usr/src/
#RUN javac /usr/src/Main.java
#FROM openjdk:11-jre
#COPY --from=builder /usr/src/Main.class .
#CMD ["java", "Main"]

FROM maven:3.6.3-jdk-11 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY /simple-api/pom.xml .
COPY /simple-api/src ./src
RUN mvn package -DskipTests

FROM openjdk:11-jre
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
