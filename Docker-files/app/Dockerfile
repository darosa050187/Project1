FROM maven:3.9.9-eclipse-temurin-21-jammy AS BUILD_IMAGE
LABEL "Project"="Vprofile" \
      "Author"="Darosa" \
      "Version"="1.0.1" \
      "environment"="Test"
# Set working directory
WORKDIR /app
# Copy only pom.xml first to leverage Docker cache for dependencies
COPY pom.xml .
RUN mvn dependenvy:go-offline
# Copy source code and build
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:21-jre-jammy
WORKDIR /app
# Add non-root user for security
RUN groupadd -r appgroup && useradd -r -g appgroup appuser
# Copy built artifact from build stage
COPY --from=BUILD_IMAGE /app/target/*.jar app.jar
# Set ownership and permissions
RUN chown -R appuser:appgroup /app
USER appuser
# Configure container healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:8080/health || exit 1
# Set environment variables
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"
# Expose port
EXPOSE 8080
# Start application