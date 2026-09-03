# Quiz Portal Microservice

A microservices-based quiz platform built with Spring Boot and Spring Cloud. Quizzes and questions live in separate services, discover each other through Eureka, and talk to each other over Feign —

## Architecture

The system is made up of four independent Spring Boot applications:

| Service | Port | Responsibility |
|---|---|---|
| **ServiceRegistry** | `8761` | Eureka server. Every other service registers here on startup and uses it to discover instances of the others. |
| **ApiGateway** | `8083` | Single public entry point. Routes incoming requests to the right downstream service using Spring Cloud Gateway, resolving instances through Eureka. |
| **QuizService** | `8081` | Owns quizzes. Create/list/fetch quizzes, and enrich each quiz with its questions by calling QuestionService. |
| **QuestionService** | `9092` | Owns questions. Create/list/fetch questions, and look up all questions belonging to a given quiz. |

```
                        ┌────────────────────┐
                        │   ServiceRegistry   │
                        │   (Eureka, 8761)    │
                        └─────────▲───────────┘
                     registers /  │  \ registers
                     discovers   │   \  discovers
                       ┌─────────┘    └─────────┐
                       │                         │
                 ┌─────▼──────┐           ┌──────▼──────┐
   Client ──────►│ ApiGateway │──lb://───►│ QuizService │
                 │   (8083)   │           │   (8081)    │
                 └─────┬──────┘           └──────┬──────┘
                       │                          │ Feign (GET /question/quiz/{id})
                       │  lb://                   ▼
                       └─────────────────►┌───────────────┐
                                           │ QuestionService│
                                           │    (9092)      │
                                           └────────────────┘
```

- The **ApiGateway** routes `/quiz/**` to `QUIZ-SERVICE` and `/question/**` to `QUESTION-SERVICE`.
- **QuizService** does not store questions itself. When a quiz is fetched, it calls **QuestionService** through a Feign client (`QuestionClient`) to pull in the questions for that quiz and attaches them to the response.

## Tech stack

- Java 17
- Spring Boot 3.5.6
- Spring Cloud 2025.0.0
  - Spring Cloud Gateway
  - Spring Cloud Netflix Eureka (server & client)
  - Spring Cloud OpenFeign
  - Spring Cloud LoadBalancer
- Spring Data JPA + MySQL
- Lombok
- Maven

## Prerequisites

- JDK 17+
- Maven 3.9+ (or just use the bundled `mvnw` wrapper in each module)
- A running MySQL instance

## Getting started

### 1. Create the database

QuizService and QuestionService both connect to the same schema:

```sql
CREATE DATABASE `quiz-portal`;
```

Both services default to these credentials in `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/quiz-portal
spring.datasource.username=root
spring.datasource.password=qwerty
```

Update them to match your local MySQL setup before running. `spring.jpa.hibernate.ddl-auto=update` is set, so tables are created/updated automatically on startup — no manual migrations needed.

### 2. Start the services, in order

Discovery has to be up first, and the gateway only makes sense once the services it routes to are registered:

```bash
# 1. Service Registry (Eureka) — must be first
cd ServiceRegistry
./mvnw spring-boot:run

# 2. Question Service
cd QuestionService
./mvnw spring-boot:run

# 3. Quiz Service (depends on QuestionService being discoverable)
cd QuizService
./mvnw spring-boot:run

# 4. API Gateway — start last
cd ApiGateway
./mvnw spring-boot:run
```

Check `http://localhost:8761` once ServiceRegistry is up — you should see `QUIZ-SERVICE`, `QUESTION-SERVICE`, and `API-GATEWAY` register there as the others come online.

## API reference

Everything is reachable through the gateway at `http://localhost:8083`, or directly against each service's own port.

### Quiz Service — `/quiz`

| Method | Endpoint | Description |
|---|---|---|
| POST | `/quiz` | Create a new quiz |
| GET | `/quiz` | List all quizzes (each one enriched with its questions) |
| GET | `/quiz/{id}` | Get a single quiz by id (enriched with its questions) |

Example — create a quiz:

```bash
curl -X POST http://localhost:8083/quiz \
  -H "Content-Type: application/json" \
  -d '{"title": "Spring Boot Basics"}'
```

### Question Service — `/question`

| Method | Endpoint | Description |
|---|---|---|
| POST | `/question` | Create a new question |
| GET | `/question` | List all questions |
| GET | `/question/{questionId}` | Get a single question by id |
| GET | `/question/quiz/{quizId}` | List all questions belonging to a given quiz |

Example — add a question to a quiz:

```bash
curl -X POST http://localhost:8083/question \
  -H "Content-Type: application/json" \
  -d '{"question": "What annotation marks a Spring Boot entry point?", "quizId": 1}'
```

## Data model

**Quiz** (`QuizService`)

| Field | Type | Notes |
|---|---|---|
| `id` | Long | Auto-generated |
| `title` | String | |
| `question` | List\<Question\> | Not persisted — populated at read time via a Feign call to QuestionService |

**Question** (`QuestionService`)

| Field | Type | Notes |
|---|---|---|
| `questionId` | Long | Auto-generated |
| `question` | String | |
| `quizId` | Long | Foreign reference to the owning quiz |

## Project structure

```
Quiz-Portal-Microservice/
├── ServiceRegistry/          # Eureka discovery server
├── ApiGateway/                # Spring Cloud Gateway routing layer
├── QuizService/
│   ├── controllers/           # REST endpoints for quizzes
│   ├── entities/              # Quiz, Question (read-only DTO for Feign responses)
│   ├── repositories/          # Spring Data JPA repository
│   └── services/               # QuizService, QuestionClient (Feign)
└── QuestionService/
    ├── controller/             # REST endpoints for questions
    ├── entities/               # Question
    ├── repostiories/           # Spring Data JPA repository
    └── service/                # QuestionService implementation
```

## Notes / known limitations

- No authentication or authorization layer yet — all endpoints are open.
- Database credentials are checked into `application.properties` for local dev convenience; swap these for environment variables or a secrets manager before deploying anywhere real.
- No quiz-submission or scoring flow yet — the current services cover authoring quizzes and questions, not taking a quiz.
