# Configuration & Externalized Settings Inventory

The configuration model is compact and relies mostly on role-specific Spring YAML files plus a small database properties file. There are no environment-specific profile files or secret-management integrations; instead, the shared jar selects one of three role configurations at startup.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| `accounts-server.yml` | Spring YAML | `src/main/resources/accounts-server.yml` | Accounts-service port, discovery client, Thymeleaf, actuator exposure |
| `web-server.yml` | Spring YAML | `src/main/resources/web-server.yml` | Web-service port, discovery client, Thymeleaf, actuator exposure |
| `registration-server.yml` | Spring YAML | `src/main/resources/registration-server.yml` | Eureka server and registration role settings |
| `db-config.properties` | Spring properties | `src/main/resources/db-config.properties` | JPA dialect hints, naming strategy, SQL logging |
| `account-controller-tests.properties` | Spring test properties | `src/main/resources/account-controller-tests.properties` | Disables Eureka for controller tests |
| `logback.xml` | Logging config | `src/main/resources/logback.xml` | Console logging and logger level setup |
| `Dockerfile` | Container config | `Dockerfile` | Base image and exposed service ports |
| Command-line arguments | Runtime input | `Main.main(String[] args)` | Selects server role and optional port override |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Maven default build | Automatic | Maintained build path for packaging and tests | Spring Boot Maven Plugin |
| Gradle default build | Manual `./gradlew` use | Legacy alternative build path called out as outdated in README | Spring Boot Gradle plugin 2.0.1.RELEASE |

No explicit Maven profiles, Gradle flavors, or environment-specific build variants are declared in the checked-in build files.

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| registration role | First CLI argument `registration` or `reg` | `registration-server.yml` | Runs Eureka on port 1111 and disables self-registration |
| accounts role | First CLI argument `accounts` | `accounts-server.yml` plus `db-config.properties` | Runs REST account service on port 2222 with discovery enabled |
| web role | First CLI argument `web` | `web-server.yml` | Runs Thymeleaf web app on port 3333 with discovery enabled |

There are no `application-dev.yml`, `application-prod.yml`, or `spring.profiles.active` definitions. Role selection is programmatic rather than profile-driven.

## Properties Inventory

### registration-server

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `spring.autoconfigure.exclude` | `org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration` | registration role | `registration-server.yml` |
| `eureka.instance.hostname` | `localhost` | registration role | `registration-server.yml` |
| `eureka.client.registerWithEureka` | `false` | registration role | `registration-server.yml` |
| `eureka.client.fetchRegistry` | `false` | registration role | `registration-server.yml` |
| `server.port` | `1111` | registration role | `registration-server.yml` |

### accounts-service

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `spring.application.name` | `accounts-service` | accounts role | `accounts-server.yml` |
| `spring.freemarker.enabled` | `false` | accounts role | `accounts-server.yml` |
| `spring.thymeleaf.cache` | `false` | accounts role | `accounts-server.yml` |
| `spring.thymeleaf.prefix` | `classpath:/accounts-server/templates/` | accounts role | `accounts-server.yml` |
| `error.path` | `/error` | accounts role | `accounts-server.yml` |
| `server.port` | `2222` | accounts role, CLI override allowed | `accounts-server.yml` and `Main.java` |
| `eureka.client.serviceUrl.defaultZone` | `http://${registration.server.hostname}:1111/eureka/` | accounts role | `accounts-server.yml` |
| `eureka.instance.leaseRenewalIntervalInSeconds` | `10` | accounts role | `accounts-server.yml` |
| `management.endpoints.web.exposure.include` | `*` | accounts role | `accounts-server.yml` |
| `spring.jpa.hibernate.ddl-auto` | `validate` | accounts role | `db-config.properties` |
| `spring.jpa.hibernate.naming_strategy` | `org.hibernate.cfg.ImprovedNamingStrategy` | accounts role | `db-config.properties` |
| `spring.jpa.database` | `H2` | accounts role | `db-config.properties` |
| `spring.jpa.show-sql` | `true` | accounts role | `db-config.properties` |

### web-service

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `spring.autoconfigure.exclude` | `org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration` | web role | `web-server.yml` |
| `spring.application.name` | `web-service` | web role | `web-server.yml` |
| `spring.freemarker.enabled` | `false` | web role | `web-server.yml` |
| `spring.thymeleaf.cache` | `false` | web role | `web-server.yml` |
| `spring.thymeleaf.prefix` | `classpath:/web-server/templates/` | web role | `web-server.yml` |
| `error.path` | `/error` | web role | `web-server.yml` |
| `server.port` | `3333` | web role, CLI override allowed | `web-server.yml` and `Main.java` |
| `eureka.client.serviceUrl.defaultZone` | `http://${registration.server.hostname}:1111/eureka/` | web role | `web-server.yml` |
| `eureka.instance.leaseRenewalIntervalInSeconds` | `5` | web role | `web-server.yml` |
| `management.endpoints.web.exposure.include` | `*` | web role | `web-server.yml` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| registration-server | `java -jar ... registration [port]` and optional `--registration.server.hostname=<IP>` | Not specified | Single instance in documented demo flow |
| accounts-service | `java -jar ... accounts [port]` and optional `--registration.server.hostname=<IP>` | Not specified | One or more instances supported by Eureka registration |
| web-service | `java -jar ... web [port]` and optional `--registration.server.hostname=<IP>` | Not specified | Single instance in documented demo flow |

No explicit `-Xms`, `-Xmx`, CPU limits, container memory limits, or scaling manifests are checked into the repository.

## Startup Dependency Chain

1. `registration-server` starts first and exposes Eureka on port 1111.
2. `accounts-service` starts next and registers itself with the registration server using `eureka.client.serviceUrl.defaultZone`.
3. `web-service` starts last so that its discovery-aware `RestTemplate` can resolve `ACCOUNTS-SERVICE` through Eureka.
4. Both application roles become observable through publicly exposed actuator endpoints once startup finishes.

The only readiness mechanism present is service registration with Eureka; there are no Docker Compose health checks, Kubernetes probes, or startup timeout policies in the repository.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| None detected in checked-in config | Not applicable | Not applicable |

### Secrets Provisioning Workflow

No secret provisioning workflow is implemented in the checked-in demo. The application uses no database passwords, API keys, key-vault references, or encrypted property sources; configuration is provided directly from YAML, properties files, and command-line arguments.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | Not applicable | Not applicable |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---:|---|
| Spring Boot parent | 2.4.2 | `pom.xml` |
| Spring Cloud BOM | 2020.0.0 | `pom.xml` |
| Spring Boot Gradle plugin | 2.0.1.RELEASE | `build.gradle` |
| Application version | 2.1.0.RELEASE | `pom.xml` |
| Docker base image | `openjdk:8-jre` | `Dockerfile` |
| Maven Wrapper | 3.6.1 | `.mvn/wrapper/maven-wrapper.properties` |
