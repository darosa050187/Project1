FROM maven:3.9.9-eclipse-temurin-21-jammy AS BUILD_IMAGE
LABEL "Project"="Vprofile" \
      "Author"="Darosa" \
      "Version"="1.0.1" \
      "environment"="Test"
RUN git clone https://github.com/darosa050187/Project1.git
RUN cd Project1 && git checkout awscontainers && mvn install

# Runtime stage
FROM tomcat:10-jdk21
RUN rm -rf /usr/local/tomcat/webapps/*
COPY --from=BUILD_IMAGE Project1/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
# Expose port
EXPOSE 8080
