FROM openjdk:11
# Build Main.java
# ...

FROM openjdk:11-jre
# Copy ressource from previous stage
COPY --from=0/usr/src/Main.class .
# Run java code with the JRE
# ...