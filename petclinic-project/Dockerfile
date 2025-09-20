# Self-contained Dockerfile that downloads and builds Spring PetClinic
FROM maven:3.8.7-eclipse-temurin-17 AS build
WORKDIR /app

# Download Spring PetClinic source code
RUN git clone https://github.com/spring-projects/spring-petclinic.git . && \
    chmod +x ./mvnw

# Build the application
RUN ./mvnw clean package -DskipTests

# Production stage
FROM openjdk:17-jdk-slim
WORKDIR /app

# Install curl for health checks
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# Copy the JAR file from build stage
COPY --from=build /app/target/*.jar app.jar

# Expose port 8080
EXPOSE 9500

# Add health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:9500/actuator/health || exit 1

# Run the application
ENTRYPOINT ["java", "-jar", "app.jar"]