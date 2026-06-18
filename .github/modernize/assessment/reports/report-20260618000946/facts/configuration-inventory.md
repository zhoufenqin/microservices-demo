# Configuration & Externalized Settings Inventory

Configuration is file-driven with per-process YAML files plus shared database settings and SQL bootstrap scripts. Runtime behavior is mostly controlled through Spring config properties and startup system properties.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| `accounts-server.yml` | Spring runtime config | `src/main/resources/accounts-server.yml` | accounts-service name, port, eureka client, actuator exposure |
| `web-server.yml` | Spring runtime config | `src/main/resources/web-server.yml` | web-service name, port, eureka client, actuator exposure |
| `registration-server.yml` | Spring runtime config | `src/main/resources/registration-server.yml` | eureka server + host/port settings |
| `db-config.properties` | Spring persistence config | `src/main/resources/db-config.properties` | JPA/Hibernate behavior for accounts-service |
| `schema.sql` | DB schema bootstrap | `src/main/resources/testdb/schema.sql` | DDL for `T_ACCOUNT` |
| `data.sql` | Seed data | `src/main/resources/testdb/data.sql` | Demo account seed rows |
| JVM system properties | Startup parameters | set in `Main`, `*Server` classes | `spring.config.name`, registration hostname, optional server port |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| default Maven build | `./mvnw ...` | Build executable jar | `spring-boot-maven-plugin` |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| registration process | `spring.config.name=registration-server` | `registration-server.yml` | port 1111, eureka server mode |
| accounts process | `spring.config.name=accounts-server` | `accounts-server.yml` + `db-config.properties` | port 2222, service name/accounts discovery + JPA behavior |
| web process | `spring.config.name=web-server` | `web-server.yml` | port 3333, service name/web discovery |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `spring.application.name` | `accounts-service` / `web-service` | process-specific | `accounts-server.yml`, `web-server.yml` |
| `server.port` | `2222`, `3333`, `1111` | process-specific | all server yml files |
| `eureka.client.serviceUrl.defaultZone` | `http://${registration.server.hostname}:1111/eureka/` | accounts, web | `accounts-server.yml`, `web-server.yml` |
| `eureka.client.registerWithEureka` | `false` | registration | `registration-server.yml` |
| `management.endpoints.web.exposure.include` | `*` | accounts, web | `accounts-server.yml`, `web-server.yml` |
| `spring.jpa.hibernate.ddl-auto` | `validate` | accounts | `db-config.properties` |
| `spring.jpa.database` | `H2` | accounts | `db-config.properties` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| registration-server | `spring.config.name=registration-server` | Not specified | 1 (default usage) |
| accounts-service | `spring.config.name=accounts-server`; optional `server.port` argument | Not specified | 1+ (README demonstrates scaling accounts instance) |
| web-service | `spring.config.name=web-server` | Not specified | 1 |

## Startup Dependency Chain

1. `registration-server` starts first and exposes Eureka registry.
2. `accounts-service` starts, reads registration host property, and registers with Eureka.
3. `web-service` starts, registers with Eureka, and resolves `ACCOUNTS-SERVICE` for downstream calls.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| None explicit | N/A | No dedicated secret store references detected |

### Secrets Provisioning Workflow

No explicit secrets provisioning workflow is defined in repository configuration. The demo uses embedded/local data without credentialed external service dependencies in checked-in config files.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N/A | N/A |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Spring Boot | 2.4.2 | `pom.xml` parent |
| Spring Cloud dependencies | 2020.0.0 | `pom.xml` dependencyManagement |
| Maven wrapper | Wrapper script present | `mvnw`, `.mvn/wrapper` |
| Java runtime target | Spring Boot 2.x compatible | managed by dependency stack |
