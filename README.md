# ActionFlow (VProfile Web App)

ActionFlow is a Java web application built with Spring MVC and JSP, packaged as a WAR and deployed on Tomcat.  
It includes user registration/login, profile updates, MySQL persistence, Memcached integration, RabbitMQ checks, and Elasticsearch operations.

## Features

- User registration and login with Spring Security + BCrypt
- User profile listing and profile update screens
- Memcached-backed user lookup (`/users/{id}`)
- RabbitMQ connectivity check page (`/user/rabbit`)
- Elasticsearch indexing/view/update/delete endpoints
- File upload endpoint for user profile images
- Docker Compose stack for app + web proxy + database + cache + message broker
- Ansible playbooks for Tomcat setup and artifact deployment

## Tech Stack

- Java 17+ (Docker images use JDK 21)
- Maven (WAR build)
- Spring Framework 6 (MVC, Security, Data JPA)
- Hibernate / JPA
- Tomcat 10
- JSP + JSTL
- MySQL 8
- Memcached
- RabbitMQ
- Elasticsearch client (7.x)
- Nginx (reverse proxy in Docker setup)

## Repository Layout

- `src/main/java` - controllers, services, repositories, utilities
- `src/main/webapp/WEB-INF` - XML configs and JSP views
- `src/main/resources` - application properties and SQL dumps
- `src/test/java` - unit/controller tests
- `Docker-files/` - Dockerfiles for app/web/db and multistage app image
- `ansible/` - provisioning and deployment playbooks
- `.github/workflows/main.yml` - CI/CD pipeline

## Prerequisites

- JDK 17 or newer
- Maven 3.9+
- Docker + Docker Compose (recommended for full stack)

## Quick Start (Docker Compose)

1. Build the WAR:

```bash
mvn clean package
```

2. Start the stack:

```bash
docker compose up --build -d
```

3. Open the app:

- Through Nginx: `http://localhost`
- Direct Tomcat app: `http://localhost:8080`

4. Stop everything:

```bash
docker compose down
```

Services started by Compose:

- `vprodb` (MySQL, port `3306`)
- `vprocache01` (Memcached, port `11211`)
- `vpromq01` (RabbitMQ, port `5672`)
- `vproapp` (Tomcat app, port `8080`)
- `vproweb` (Nginx, port `80`)

## Local Run (Without Docker)

1. Start dependencies manually:
- MySQL 8
- Memcached
- RabbitMQ
- Elasticsearch (optional; needed for Elasticsearch endpoints)

2. Create/import DB:

```bash
mysql -u <user> -p -e "CREATE DATABASE IF NOT EXISTS accounts;"
mysql -u <user> -p accounts < src/main/resources/db_backup.sql
```

3. Update `src/main/resources/application.properties` with local hostnames/ports/credentials.

4. Run app:

```bash
mvn jetty:run
```

Then open `http://localhost:8080`.

## Key Endpoints

- `GET /` - login page
- `GET/POST /registration` - sign up
- `GET /welcome` - authenticated welcome page
- `GET /users` - list users
- `GET /users/{id}` - user by id (with Memcached flow)
- `GET /user/{username}` - profile edit page
- `POST /user/{username}` - profile update
- `GET /upload` - upload UI
- `POST /uploadFile` - upload profile image
- `GET /user/rabbit` - RabbitMQ status page
- `GET /user/elasticsearch` - bulk index users in Elasticsearch
- `GET /rest/users/view/{id}` - read user doc from Elasticsearch
- `GET /rest/users/update/{id}` - update user doc in Elasticsearch
- `GET /rest/users/delete/{id}` - delete user doc in Elasticsearch

## Build, Test, and Checks

```bash
mvn clean package
mvn test
mvn checkstyle:check
```

## CI/CD

GitHub Actions workflow (`.github/workflows/main.yml`) includes:

- Build (`mvn install`)
- Test + Checkstyle
- Trivy filesystem scan
- Docker image build/push to Amazon ECR (on `main`, with AWS secrets/vars configured)

## Notes

- SQL dumps are available in:
  - `src/main/resources/db_backup.sql`
  - `src/main/resources/accountsdb.sql`
- Included dump users are stored with BCrypt hashes (plain-text passwords are not included).
- Elasticsearch client code uses HTTP; ensure `elasticsearch.host`/`elasticsearch.port` match your HTTP endpoint (commonly port `9200`).

