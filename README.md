# Overview
This is the main repository for an example Spring Boot/Spring Cloud project.
It consists of four microservices described in detail below.
It is designed to be run either from prebuilt container images (docker-compose.yml) or built locally for development (docker-compose.dev.yml).

### Services in this repo 
1. [eureka-server](https://github.com/Milozap/tec-eureka-server) — service registry: Spring Cloud Netflix Eureka
2. [storage-service](https://github.com/Milozap/tec-storage-service) — persistence: H2 + Flyway
3. [api-service](https://github.com/Milozap/tec-api-service) — REST API aggregating and calling storage-service, JWT protected
4. [gateway-service](https://github.com/Milozap/tec-gateway-service) — service routing to api-service with simple rate limiting

### Tech stack
- Language: Java Java 25 
- Framework: Spring Boot 3.5.x, Spring Cloud 2025.0.0
- Build: Gradle
- Datastore: H2 for storage-service
- Packaging/Runtime: Docker and Docker Compose

### Used ports
- gateway-service: 8080
- storage-service: 8081
- api-service: 8082
- eureka-server: 8761

# Requirements
- Docker and Docker Compose 
- Git
- For Development:
  - [JDK 25](https://www.oracle.com/java/technologies/downloads/#jdk25-linux)
  - Gradle
- JWT Token for testing: instructions below

### Files
- .env.example — sample environment; copy to .env and adjust
- docker-compose.yml — uses prebuilt images from a registry. Each repo creates a new image on push to the main branch.
- docker-compose.dev.yml — builds images from local sources for development

# Getting started
1) Clone the repository
   ```shell
   git clone https://github.com/Milozap/tec-architecture
   cd tec-architecture
   ```

2) Prepare environment
   Copy .env.example to .env and update values as needed (make sure to change JWT_SECRET to non-default):
   ```shell
   cp .env.example .env
   ```
## JWT Token generation
To use the api and `/movies` endpoints JWT token is required. To generate the token, websites like https://www.jwt.io/ can be used.
Leave the headers as:
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```
Set the payload as, for example: 
```json
{
  "sub": "test-user",
  "scope": "movies.read movies.write",
  "iat": 1731950000,
  "exp": 4731953600
}
```
And set the JWT Secret as secret from .env ("dev-jwt-secret-dev-jwt-secret-dev-jwt-secret-dev-jwt-secret" by default)
```json
dev-jwt-secret-dev-jwt-secret-dev-jwt-secret-dev-jwt-secret
```

Click generate example and copy the jwt token. Then use it with `Authorization: Bearer <token>` header to access api routes.

## Dev helper script (multi-repo note)
To run the local build with `docker-compose.dev.yml` all the repos are required. To easily get them run:
```shell
echo "Cloning Eureka server repository"
git clone https://github.com/Milozap/tec-eureka-server.git

echo "Cloning Storage Service repository"
git clone https://github.com/Milozap/tec-storage-service.git

echo "Cloning Api Service repository"
git clone https://github.com/Milozap/tec-api-service.git

echo "Cloning Gateway Service repository"
git clone https://github.com/Milozap/tec-gateway-service.git
```

## Run with prebuilt images
```shell
docker compose --env-file .env -f docker-compose.yml up -d
```

## Update prebuilt images
```shell
docker compose pull
```

## Run with local build - for development
Make sure to get other repos before running this command. Otherwise it will fail.
```shell
docker compose --env-file .env -f docker-compose.dev.yml up -d --build
```

### Local development without Docker (per service)
Each service is a standalone Spring Boot app. From the service directory you can run:
```shell
./gradlew bootRun
```

### Swagger
Swagger docs available at URL: http://localhost:8080/swagger-ui/index.html

### Configuration notes
- Services read EUREKA_URL from the environment (defaults to http://localhost:8761/eureka when running without Docker).
- api-service reads JWT_SECRET from the environment; a default is provided only for development purposes. **Do not** use defaults in production.

### Health and management endpoints
- Actuator endpoints are enabled (health, info; additional endpoints in api-service). Access may be limited depending on the environment and security configuration.
