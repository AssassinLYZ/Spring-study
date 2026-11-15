# imooc-hire-dev

A **recruitment platform** built as a Spring Cloud microservices study project, used to practice common microservice techniques.

Three types of users: job seekers (APP), company HRs (SaaS), and platform admins (back office).

## Tech Stack

- Spring Boot 2.6.11 / Spring Cloud 2021.0.4 / Spring Cloud Alibaba 2021.0.1.0
- JDK 1.8, Maven multi-module
- Nacos (service registry and config center), Spring Cloud Gateway, OpenFeign
- MyBatis-Plus, PageHelper, MySQL, MongoDB
- Redis, Redisson, Caffeine, Zookeeper (Curator)
- RabbitMQ, Seata, Canal
- MinIO / Aliyun OSS, Knife4j

## Modules

### Shared modules (not run on their own)

| Module | Description |
|---|---|
| `hire-common` | Utilities, enums, unified response wrapper, JWT, etc. |
| `hire-pojo` | Entities and BO / VO / CO / MO classes |
| `hire-api` | Shared web dependencies: Feign clients, interceptors, Redisson, ZK lock, MQ config, thread pool |

### Microservices

| Service | Port | Description |
|---|---|---|
| `gateway-8000` | 8000 | API gateway: routing, JWT verification, IP rate limiting, CORS |
| `auth-service-8111` | 8111 | Authentication: mobile login, HR QR-code login, admin login |
| `service-user-7001` | 7001 | User profiles, binding HR to a company, admin account management |
| `service-company-6001` | 6001 | Company info, verification documents, company review |
| `service-work-3001` | 3001 | Resumes, job postings, reports |
| `server-resource-4001` | 4001 | Industries, job types, data dictionary, system parameters |
| `service-file-5001` | 5001 | File upload (avatars, logos, business licenses, etc.) |

### Demo modules (not microservices)

`rabbit-producer` and `rabbit-consumer` are examples of the native RabbitMQ API (Work Queues, Pub/Sub, Routing, Topics). They have no application entry point and are unrelated to the business logic.

## Request Flow

```
Client -> gateway-8000 (JWT check / IP rate limit)
            |- /u /userinfo /admininfo   -> user-service
            |- /c /company               -> company-service
            |- /a /passport /saas /admin -> auth-service
            |- /industry /dataDict ...   -> resource-service
            |- /w /resume /job /report   -> work-service
            `- /file /static             -> file-service
```

After verifying the JWT, the gateway puts the user info into a request header and forwards it. Downstream services do not verify the token again.

## Topics Covered

- Unified authentication and IP rate limiting at the gateway
- Distributed locks: Redisson (fair lock, read/write lock, semaphore) and Zookeeper
- Distributed transactions: Seata, and eventual consistency with a local message table + MQ
- RabbitMQ: manual ack, delay queues, dead-letter queues
- Canal listening to MySQL binlog to sync the data dictionary and system parameters
- Caffeine local cache

## Running

Build:

```bash
mvn clean install -DskipTests
```

Running the services requires middleware: Nacos, MySQL (database `imooc-hire-dev`), Redis, Zookeeper, RabbitMQ, MongoDB and Seata Server. The file service also needs MinIO or OSS. Update the middleware addresses in each service's `application-dev.yml` to match your environment.

The gateway reads `jwt_config.yaml` from the Nacos config center, so create it before starting.

## Notes

This is a study project. It keeps many commented-out alternatives and simulated logic used for demonstrations (for example a fixed SMS code and deliberate exceptions), so it is **not suitable for production use**.
