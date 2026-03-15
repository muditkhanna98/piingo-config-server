# Piingo Config Server

`piingo-config-server` is a Spring Cloud Config Server for Piingo services.

It serves externalized configuration from a separate Git repository so services like `user-service` can load their config at startup instead of keeping environment-specific values inside the application repo.

## Tech Stack

- Java 25
- Spring Boot 4.0.3
- Spring Cloud Config Server
- Maven Wrapper

## What This Server Does

- Runs on port `8888`
- Connects to a Git-based config repository
- Serves configuration files to client applications such as `user-service`
- Exposes `health` and `info` actuator endpoints

## Configuration

The server reads its Git backend settings from environment variables:

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `CONFIG_REPO_URI` | Yes | `<url-to-repo>` | Git repository that stores application config files |
| `CONFIG_REPO_BRANCH` | No | `main` | Git branch the config server should read from |

## Expected Config Repo Structure

Your separate config repository can look like this:

```text
piingo-user-service-config/
  user-service.yml
  user-service-dev.yml
  user-service-prod.yml
```

Example `user-service.yml`:

```yaml
server:
  port: 8081

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/piingo_user
    username: root
    password: secret

custom:
  welcome-message: Hello from config server
```

## Running Locally

Set the config repo URL before starting the server:

```bash
export CONFIG_REPO_URI=https://github.com/<your-org>/<your-config-repo>.git
export CONFIG_REPO_BRANCH=main
./mvnw spring-boot:run
```

The server will start at:

```text
http://localhost:8888
```

## Useful Endpoints

Fetch default config for `user-service`:

```text
GET /user-service/default
```

Fetch a specific profile:

```text
GET /user-service/dev
GET /user-service/prod
```

Health endpoint:

```text
GET /actuator/health
```

## user-service Client Setup

In `user-service`, point Spring to this config server:

```properties
spring.application.name=user-service
spring.config.import=optional:configserver:http://localhost:8888
```

With that in place, `user-service` will request its configuration from this server during startup.

## Build And Test

Run tests:

```bash
./mvnw clean test
```

## Notes

- `clone-on-start` is disabled, so the app can boot even before the real Git repo is configured.
- `force-pull` is enabled so the server refreshes from the remote repository when serving config.
- The checked-in `config-repo/` folder is only leftover local scaffolding from the initial native setup and is no longer used by the active server configuration.
