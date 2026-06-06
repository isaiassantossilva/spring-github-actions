# spring-github-actions

A study project demonstrating CI/CD with **GitHub Actions** for a containerized
full-stack application: a **Spring Boot** backend and a **React + Vite**
frontend. Each app has its own workflow that runs tests, builds a Docker image,
pushes it to a container registry, and triggers a deploy.

## Project structure

```
.
├── .github/workflows/
│   ├── backend.yml                  # Backend CI/CD pipeline
│   └── frontend.yml                 # Frontend CI/CD pipeline
├── spring-github-actions-backend/   # Spring Boot (Java 25, Gradle) API
└── spring-github-actions-frontend/  # React 19 + Vite (TypeScript) SPA
```

## Tech stack

| Area      | Stack                                                            |
| --------- | --------------------------------------------------------------- |
| Backend   | Java 25, Spring Boot 4, Spring Web MVC, Actuator, Gradle (Kotlin DSL) |
| Frontend  | React 19, TypeScript, Vite, ESLint, served by Nginx             |
| CI/CD     | GitHub Actions, Docker Buildx, multi-stage Dockerfiles          |

## Backend

Spring Boot service exposing Actuator endpoints (health probe at
`/actuator/health`), packaged as a multi-stage Docker image based on
`eclipse-temurin` and running as a non-root user on port `8080`.

```bash
cd spring-github-actions-backend

./gradlew test          # run tests
./gradlew bootRun       # run locally (http://localhost:8080)
./gradlew bootJar       # build the executable jar

# Build and run the container
docker build -t spring-github-actions-backend .
docker run --rm -p 8080:8080 spring-github-actions-backend
```

## Frontend

React single-page app built with Vite and served as static assets by Nginx
(with SPA fallback to `index.html`) on port `80`.

```bash
cd spring-github-actions-frontend

npm ci                  # install dependencies
npm run dev             # dev server (http://localhost:5173)
npm run lint            # lint
npm run build           # typecheck + production bundle

# Build and run the container
docker build -t spring-github-actions-frontend .
docker run --rm -p 8080:80 spring-github-actions-frontend
```

## CI/CD pipelines

Both workflows live in [`.github/workflows/`](.github/workflows) and follow the
same three-stage pattern. Each is path-filtered, so the backend pipeline only
runs when backend files change, and likewise for the frontend.

**Triggers:** push to `main`, push of a `v*` tag, or manual `workflow_dispatch`.

| Stage     | Backend                                  | Frontend                                |
| --------- | ---------------------------------------- | --------------------------------------- |
| `test`    | JDK 25 + Gradle → `./gradlew test`       | Node 22 → `npm run lint` + `npm run build` |
| `publish` | Docker Buildx → build & push image       | Docker Buildx → build & push image      |
| `deploy`  | Trigger deploy webhook                   | Trigger deploy webhook                  |

Image tags are derived with `docker/metadata-action` (branch, semver from
`v*` tags, short SHA, and `latest` on the default branch). Build layers are
cached via GitHub Actions cache (`type=gha`).

### Required secrets

| Secret                       | Used for                          |
| ---------------------------- | --------------------------------- |
| `CONTAINER_REGISTRY_USERNAME` | Container registry login + image namespace |
| `CONTAINER_REGISTRY_PASSWORD` | Container registry login          |

> The registry, webhook URL, and deploy token in the workflows are placeholder
> values (`registry.example.net`, `webhook.example.net`, `token-example`) and
> should be replaced for a real deployment.

## License

This is a learning/study project and is provided as-is.
