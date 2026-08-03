# Quiz Portal Microservice

A Spring Boot / Spring Cloud microservices project for managing quizzes and questions, with service discovery and API gateway routing.

## Architecture

The system is composed of four independent Spring Boot services:

| Service | Port | Responsibility |
|---|---|---|
| **ServiceRegistry** | `8761` | Eureka service registry used by all other services for discovery. |
| **ApiGateway** | `8083` | Single entry point that routes external requests to the appropriate downstream service via Spring Cloud Gateway. |
| **QuizService** | `8081` | Manages quizzes (create, list, fetch by id). |
| **QuestionService** | `9092` | Manages questions belonging to a quiz (create, list, fetch by id, fetch by quiz). |

```
Client → ApiGateway (8083) → QuizService (8081) / QuestionService (9092)
                ↑                        ↑                  ↑
                └──────── ServiceRegistry (8761) ────────────┘
```

The `ApiGateway` routes requests prefixed with `/quiz/**` to `QUIZ-SERVICE` and `/question/**` to `QUESTION-SERVICE`, resolving instances through Eureka.

## Tech Stack

- Java 17
- Spring Boot 3.5.6
- Spring Cloud 2025.0.0
- Spring Cloud Gateway
- Spring Cloud Netflix Eureka (server & client)
- Spring Data JPA
- MySQL
- Maven

## Prerequisites

- JDK 17+
- Maven 3.9+ (or use the included `mvnw` wrapper)
- MySQL running locally with a `quiz-portal` database

## Getting Started

### 1. Database

Create the database used by `QuizService` and `QuestionService`:

```sql
CREATE DATABASE `quiz-portal`;
```

Both services default to:

```
spring.datasource.url=jdbc:mysql://localhost:3306/quiz-portal
spring.datasource.username=root
spring.datasource.password=qwerty
```

Update these in each service's `src/main/resources/application.properties` to match your local MySQL credentials.

### 2. Run the services

Start the services in the following order, each from its own directory:

```bash
# 1. Service Registry (Eureka)
cd ServiceRegistry
./mvnw spring-boot:run

# 2. Quiz Service
cd QuizService
./mvnw spring-boot:run

# 3. Question Service
cd QuestionService
./mvnw spring-boot:run

# 4. API Gateway
cd ApiGateway
./mvnw spring-boot:run
```

Once running, the Eureka dashboard is available at `http://localhost:8761`.

## API Endpoints

All requests can go through the API Gateway at `http://localhost:8083`, or directly to each service.

### Quiz Service (`/quiz`)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/quiz` | Create a new quiz |
| GET | `/quiz` | List all quizzes |
| GET | `/quiz/{id}` | Get a quiz by id |

### Question Service (`/question`)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/question` | Create a new question |
| GET | `/question` | List all questions |
| GET | `/question/{questionId}` | Get a question by id |
| GET | `/question/quiz/{quizId}` | List questions for a given quiz |

## Project Structure

```
Quiz-Portal-Microservice/
├── ServiceRegistry/    # Eureka discovery server
├── ApiGateway/         # Spring Cloud Gateway routing
├── QuizService/        # Quiz management service
└── QuestionService/    # Question management service
```
