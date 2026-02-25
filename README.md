# 🚀 6-Month Backend & Infrastructure Engineering Roadmap
### Day-by-Day | Database → API → Auth → Docker → Cloud → Scale

> **Goal:** Become a production-ready Backend / AI Infrastructure Engineer in 180 days.
> No fluff. No theory overload. Pure engineering — databases, APIs, auth, containers, cloud, and scale.

---

## 📌 How to Use This Roadmap

- Follow the months **in order** — each builds on the last
- Each day has a **clear single target** — complete it before moving on
- Weekends = review + mini-project (don't skip them)
- Track your progress by checking off each day
- Push **every project** to GitHub as you build it

---

## 🗓️ MONTH 1 — Database Design & Engineering
> **Theme:** Master PostgreSQL, schema design, indexing, and querying before touching any API code.

---

### WEEK 1 — PostgreSQL Fundamentals

| Day | Target |
|-----|--------|
| **Day 1** | Install PostgreSQL locally. Learn `psql` CLI basics — connect, list databases, list tables, quit. Create your first database. |
| **Day 2** | Learn data types: `INTEGER`, `VARCHAR`, `TEXT`, `BOOLEAN`, `TIMESTAMP`, `UUID`, `DECIMAL`. Create a `users` table with proper types. |
| **Day 3** | Learn `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`. Practice adding/removing columns. Understand `NOT NULL`, `DEFAULT`, `UNIQUE`. |
| **Day 4** | Learn `INSERT INTO`, `SELECT`, `UPDATE`, `DELETE`. Practice all four on your `users` table. Understand `WHERE` clauses. |
| **Day 5** | Learn Primary Keys (`PRIMARY KEY`), Foreign Keys (`REFERENCES`), and `ON DELETE CASCADE`. Create a `posts` table linked to `users`. |
| **Day 6** | **Weekend Review:** Draw an ER diagram of your `users` + `posts` schema. Add 3 more tables (comments, likes, categories). Write SQL to insert test data. |
| **Day 7** | **Mini Project:** Design and build a simple blog database schema with at least 4 tables. Write `INSERT` statements to populate 20 rows of realistic data. |

---

### WEEK 2 — Advanced Querying & Relationships

| Day | Target |
|-----|--------|
| **Day 8** | Learn `JOIN` types: `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN`. Write queries joining `users` + `posts`. |
| **Day 9** | Learn aggregate functions: `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`. Use `GROUP BY` and `HAVING`. Count posts per user. |
| **Day 10** | Learn subqueries — correlated and non-correlated. Find users who have more than 5 posts using a subquery. |
| **Day 11** | Learn `ORDER BY`, `LIMIT`, `OFFSET`. Implement manual pagination — page 1, page 2, page 3 of posts. |
| **Day 12** | Learn `DISTINCT`, `IN`, `NOT IN`, `BETWEEN`, `LIKE`, `ILIKE`. Write 10 different filter queries on your blog data. |
| **Day 13** | **Weekend Review:** Write 15 complex queries combining JOINs + aggregates + filters. Debug any slow-feeling queries. |
| **Day 14** | **Mini Project:** Build a "reporting database" — write queries that produce: top 5 most active users, most commented posts this week, average post length by category. |

---

### WEEK 3 — Indexing, Performance & Transactions

| Day | Target |
|-----|--------|
| **Day 15** | Learn what indexes are and why they matter. Create `CREATE INDEX` on `email` and `created_at` columns. Use `EXPLAIN` to see query plans. |
| **Day 16** | Learn `EXPLAIN ANALYZE`. Run a slow query without index, then with. Compare execution times. Understand `Seq Scan` vs `Index Scan`. |
| **Day 17** | Learn composite indexes. When to use them. Create a composite index on `(user_id, created_at)` for posts and test it. |
| **Day 18** | Learn database transactions — `BEGIN`, `COMMIT`, `ROLLBACK`. Understand ACID properties. Write a transaction that transfers "credits" between two users atomically. |
| **Day 19** | Learn `SAVEPOINT` and partial rollbacks. Practice scenarios where a multi-step operation partially fails and you roll back to a savepoint. |
| **Day 20** | Learn Connection Pooling concepts. Install and configure **PgBouncer** or understand `max_connections` in PostgreSQL config. |
| **Day 21** | **Weekend Mini Project:** Optimize the blog database — add proper indexes, run `EXPLAIN ANALYZE` on 5 queries before and after, document the performance improvement. |

---

### WEEK 4 — SQLAlchemy, Alembic & Python Integration

| Day | Target |
|-----|--------|
| **Day 22** | Install SQLAlchemy. Understand ORM vs Core. Define your `User` and `Post` models using `DeclarativeBase`. |
| **Day 23** | Learn SQLAlchemy sessions — `Session`, `add()`, `commit()`, `rollback()`, `close()`. Insert and query data using the ORM. |
| **Day 24** | Learn SQLAlchemy relationships — `relationship()`, `back_populates`, `lazy` loading vs `joinedload`. Query a user and their posts in one call. |
| **Day 25** | Learn SQLAlchemy filtering — `.filter()`, `.filter_by()`, `.order_by()`, `.limit()`, `.offset()`. Implement pagination in Python. |
| **Day 26** | Install **Alembic**. Initialize it in your project. Understand `alembic.ini` and `env.py`. Generate your first migration from existing models. |
| **Day 27** | Practice Alembic migrations — `alembic revision --autogenerate`, `alembic upgrade head`, `alembic downgrade -1`. Add a new column via migration. |
| **Day 28** | **Month 1 Capstone Project:** Build a fully designed **E-Commerce Database** with tables: `users`, `products`, `categories`, `orders`, `order_items`, `reviews`. Write all SQLAlchemy models, Alembic migrations, and 10 complex queries. Push to GitHub. |

---

## 🗓️ MONTH 2 — API Engineering with FastAPI
> **Theme:** Build clean, validated, versioned, production-ready REST APIs on top of your database.

---

### WEEK 5 — FastAPI Fundamentals

| Day | Target |
|-----|--------|
| **Day 29** | Install FastAPI + Uvicorn. Create your first `main.py`. Run the server. Visit `/docs` (Swagger UI) and `/redoc`. Understand automatic docs generation. |
| **Day 30** | Learn path parameters and query parameters. Create routes: `GET /users/{user_id}` and `GET /users?page=1&limit=10`. |
| **Day 31** | Learn **Pydantic** — define request and response schemas. Create `UserCreate`, `UserResponse`, `UserUpdate` models. Understand field validation. |
| **Day 32** | Learn HTTP methods properly — `GET`, `POST`, `PUT`, `PATCH`, `DELETE`. Build a full CRUD API for `users` using in-memory data (no DB yet). |
| **Day 33** | Learn status codes — `200`, `201`, `204`, `400`, `401`, `403`, `404`, `422`, `500`. Return correct codes from every endpoint. |
| **Day 34** | Learn FastAPI dependency injection — `Depends()`. Create a reusable `get_db` dependency that opens and closes a DB session. |
| **Day 35** | **Weekend Mini Project:** Connect FastAPI to your Month 1 PostgreSQL database via SQLAlchemy. Build full CRUD for `users` — create, read, update, delete. Test in Swagger UI. |

---

### WEEK 6 — Database-Connected APIs

| Day | Target |
|-----|--------|
| **Day 36** | Add `posts` CRUD endpoints. Practice foreign key relationships — create a post for a specific user: `POST /users/{user_id}/posts`. |
| **Day 37** | Learn **response models** — use `response_model=` in FastAPI. Create `PostResponse` that excludes sensitive fields. |
| **Day 38** | Implement **pagination** properly — `skip` and `limit` query params. Return `total`, `page`, `per_page`, `data` in your list responses. |
| **Day 39** | Implement **filtering** — filter posts by `category_id`, `user_id`, `created_after` date. Build dynamic SQLAlchemy filters. |
| **Day 40** | Learn **global exception handling** — `@app.exception_handler()`. Create custom `AppException` class. Return consistent error response format. |
| **Day 41** | Learn **request validation errors** — handle Pydantic `ValidationError`, return `422` with clear field-level error messages. |
| **Day 42** | **Weekend Mini Project:** Build a complete **Posts API** — CRUD for posts, nested routes for comments, pagination, filtering, and consistent error responses. |

---

### WEEK 7 — API Design Best Practices

| Day | Target |
|-----|--------|
| **Day 43** | Learn **API versioning** — implement `/api/v1/` prefix using `APIRouter`. Set up project folder structure: `routers/`, `schemas/`, `models/`, `services/`. |
| **Day 44** | Learn **service layer pattern** — move business logic out of route handlers into `services/user_service.py`, `services/post_service.py`. |
| **Day 45** | Learn environment configuration — use `python-dotenv` and Pydantic `BaseSettings`. Load `DATABASE_URL`, `SECRET_KEY`, `DEBUG` from `.env`. |
| **Day 46** | Learn **middleware** — add request logging middleware that logs method, path, status code, and response time for every request. |
| **Day 47** | Learn **CORS** — configure `CORSMiddleware`. Understand origins, methods, headers. Set up for local frontend dev. |
| **Day 48** | Learn **file uploads** — `UploadFile`, `File()`. Create an endpoint to upload a profile picture. Save it to local disk for now. |
| **Day 49** | **Weekend Mini Project:** Refactor your entire project to use proper folder structure, service layer, versioning, and environment config. |

---

### WEEK 8 — Testing & API Quality

| Day | Target |
|-----|--------|
| **Day 50** | Learn `pytest`. Write your first test. Understand `assert`, fixtures, and test discovery. Run `pytest -v`. |
| **Day 51** | Learn FastAPI `TestClient`. Write integration tests for your `users` endpoints — test create, read, update, delete. |
| **Day 52** | Learn test database setup — use a separate test DB or SQLite in-memory. Use `pytest` fixtures to create and tear down the DB. |
| **Day 53** | Write tests for error cases — test `404 Not Found`, `422 Validation Error`, `400 Bad Request` responses. |
| **Day 54** | Learn `pytest-cov` for code coverage. Aim for 80%+ coverage on your routers and services. |
| **Day 55** | Learn **API documentation** best practices — add `summary`, `description`, `tags`, `response_description` to all endpoints. |
| **Day 56** | **Month 2 Capstone:** Build a full **Blog Platform API** — users, posts, comments, categories, pagination, filtering, versioning, error handling, 80%+ test coverage. Push to GitHub with a proper README. |

---

## 🗓️ MONTH 3 — Authentication, Security & Access Control
> **Theme:** Secure your APIs with JWT auth, RBAC, API keys, and rate limiting.

---

### WEEK 9 — JWT Authentication

| Day | Target |
|-----|--------|
| **Day 57** | Understand authentication vs authorization. Learn how JWT works — header, payload, signature. Decode a JWT at `jwt.io`. |
| **Day 58** | Install `python-jose` and `passlib`. Hash a password with bcrypt. Verify a hashed password. Never store plaintext passwords. |
| **Day 59** | Build `POST /auth/register` — accept email + password, hash password, store user, return user data (not password). |
| **Day 60** | Build `POST /auth/login` — verify email + password, generate JWT access token, return it. Understand `exp` claim. |
| **Day 61** | Build `get_current_user` dependency — extract Bearer token from `Authorization` header, decode JWT, return user. |
| **Day 62** | Protect routes with `current_user = Depends(get_current_user)`. Test that unauthenticated requests get `401`. |
| **Day 63** | **Weekend Mini Project:** Implement **refresh tokens** — short-lived access token (15 min) + long-lived refresh token (7 days). Store refresh tokens in DB. Build `POST /auth/refresh`. |

---

### WEEK 10 — Authorization & Role-Based Access Control

| Day | Target |
|-----|--------|
| **Day 64** | Add `role` field to `users` table — values: `user`, `admin`, `moderator`. Write Alembic migration. |
| **Day 65** | Build `require_role()` dependency factory — `Depends(require_role("admin"))`. Return `403` if role doesn't match. |
| **Day 66** | Implement resource ownership — users can only edit/delete **their own** posts. Return `403` if user_id doesn't match. |
| **Day 67** | Build admin-only endpoints — `GET /admin/users` (list all users), `DELETE /admin/users/{id}` (delete any user). |
| **Day 68** | Learn **OAuth2 password flow** — understand how FastAPI's `OAuth2PasswordRequestForm` works. Refactor your login endpoint to use it. |
| **Day 69** | Add `is_active` and `is_verified` flags to users. Block inactive users from logging in. Return clear error messages. |
| **Day 70** | **Weekend Mini Project:** Build a full **permission matrix** — users can CRUD their own posts, moderators can delete any post, admins can do everything. Test all permission scenarios. |

---

### WEEK 11 — API Security & Rate Limiting

| Day | Target |
|-----|--------|
| **Day 71** | Install Redis locally. Learn basic Redis commands — `SET`, `GET`, `DEL`, `EXPIRE`, `TTL`, `INCR`. Connect to Redis from Python using `redis-py`. |
| **Day 72** | Implement **rate limiting** using Redis — allow 100 requests per minute per IP. Return `429 Too Many Requests` when exceeded. |
| **Day 73** | Implement **per-user rate limiting** — authenticated users get higher limits (500/min) vs anonymous (20/min). |
| **Day 74** | Build an **API key system** — generate random API keys, hash and store them, allow requests via `X-API-Key` header as an alternative to JWT. |
| **Day 75** | Learn **security headers** — add `X-Content-Type-Options`, `X-Frame-Options`, `Strict-Transport-Security` via middleware. |
| **Day 76** | Learn SQL injection prevention — understand why SQLAlchemy ORM protects you. Write a raw query safely using `text()` with bound parameters. Learn about input sanitization. |
| **Day 77** | **Weekend Mini Project:** Security audit your Blog API — test for missing auth, over-permissive endpoints, missing rate limits. Fix all issues found. |

---

### WEEK 12 — Email Verification & Password Reset

| Day | Target |
|-----|--------|
| **Day 78** | Build `POST /auth/verify-email` flow — generate a time-limited verification token (UUID or signed JWT), store in DB. |
| **Day 79** | Build `POST /auth/forgot-password` — accept email, generate reset token, store with expiry. Return success even if email not found (prevents user enumeration). |
| **Day 80** | Build `POST /auth/reset-password` — verify token, check expiry, update password, invalidate token. |
| **Day 81** | Integrate **email sending** — use `smtplib` or `fastapi-mail`. Send verification and reset emails (use Mailtrap for testing). |
| **Day 82** | Add **token blacklisting** — on logout, store the JWT `jti` in Redis with TTL equal to remaining token lifetime. Reject blacklisted tokens. |
| **Day 83** | Write tests for all auth flows — register, login, refresh, logout, forgot password, reset password. Cover happy paths and edge cases. |
| **Day 84** | **Month 3 Capstone:** Full **Secure Auth System** — JWT + refresh tokens, RBAC, API keys, rate limiting, email verification, password reset, token blacklisting. Push to GitHub. |

---

## 🗓️ MONTH 4 — Docker, Caching & Background Tasks
> **Theme:** Containerize everything, add caching for speed, and process jobs asynchronously.

---

### WEEK 13 — Docker Fundamentals

| Day | Target |
|-----|--------|
| **Day 85** | Install Docker. Learn core concepts: images, containers, layers, volumes, networks. Run `docker run hello-world`. Pull and run `postgres:15` container. |
| **Day 86** | Write your first `Dockerfile` for your FastAPI app. Understand `FROM`, `WORKDIR`, `COPY`, `RUN`, `EXPOSE`, `CMD`. Build and run it. |
| **Day 87** | Learn `docker-compose.yml`. Define services: `api` (your FastAPI app) and `db` (PostgreSQL). Connect them via service name DNS. |
| **Day 88** | Add environment variables to docker-compose using `.env` file. Add a `volumes` mount for PostgreSQL data persistence. |
| **Day 89** | Learn **multi-stage builds** — stage 1: install all deps and build, stage 2: copy only what's needed. Reduce image size significantly. |
| **Day 90** | Learn Docker networking — `bridge`, `host` networks. Learn how containers communicate. Add a `redis` service to your compose file. |
| **Day 91** | **Weekend Mini Project:** Full `docker-compose up` that starts: FastAPI app + PostgreSQL + Redis + runs Alembic migrations on startup. One command = full working system. |

---

### WEEK 14 — Redis Caching Strategies

| Day | Target |
|-----|--------|
| **Day 92** | Understand caching theory — cache-aside, write-through, write-behind. When to cache vs when not to. |
| **Day 93** | Implement **cache-aside** pattern — before querying DB, check Redis. If hit, return cached. If miss, query DB, store in Redis, return. |
| **Day 94** | Cache a `GET /posts/{id}` endpoint. Set TTL of 5 minutes. Test cache hits vs misses using Redis `MONITOR`. |
| **Day 95** | Implement **cache invalidation** — when a post is updated or deleted, delete its cache key. When a new post is created, invalidate the post list cache. |
| **Day 96** | Cache **list endpoints** with pagination — cache `GET /posts?page=1&limit=10`. Use the full query string as cache key. |
| **Day 97** | Implement **cache stampede prevention** — when a popular key expires, multiple requests hit DB simultaneously. Use a mutex lock pattern with Redis `SETNX`. |
| **Day 98** | **Weekend Mini Project:** Add caching to your entire Blog API. Measure response times before and after with `ab` (Apache Bench) or `wrk`. Document the improvement. |

---

### WEEK 15 — Background Tasks & Celery

| Day | Target |
|-----|--------|
| **Day 99** | Understand why background tasks exist — email sending, image processing, report generation. Learn FastAPI's built-in `BackgroundTasks`. |
| **Day 100** | Use `BackgroundTasks` to send a welcome email after registration without blocking the response. |
| **Day 101** | Install **Celery** with Redis as broker. Create your first Celery task. Run a Celery worker. |
| **Day 102** | Move email sending to a Celery task. Call `send_welcome_email.delay(user_id)` from your route. |
| **Day 103** | Learn **Celery task states** — `PENDING`, `STARTED`, `SUCCESS`, `FAILURE`. Build `GET /tasks/{task_id}/status` endpoint to check task progress. |
| **Day 104** | Learn **Celery beat** — scheduled tasks. Schedule a daily task that sends a "digest email" to all users (log it for now). |
| **Day 105** | **Weekend Mini Project:** Add Celery to your docker-compose. Services: `api`, `db`, `redis`, `celery-worker`, `celery-beat`. All started with one command. |

---

### WEEK 16 — File Storage & Advanced Docker

| Day | Target |
|-----|--------|
| **Day 106** | Learn **MinIO** — self-hosted S3-compatible object storage. Add a `minio` service to docker-compose. Create a bucket. |
| **Day 107** | Build a file upload endpoint — accept image/pdf, upload to MinIO using `boto3` client, store the URL in PostgreSQL. |
| **Day 108** | Build `GET /files/{file_id}` — generate a presigned URL from MinIO with 1-hour expiry. Return it to the client. |
| **Day 109** | Add image processing background task — when a profile picture is uploaded, use Celery to resize it to 3 sizes (thumbnail, medium, large). |
| **Day 110** | Learn **Docker health checks** — add `HEALTHCHECK` to your Dockerfile and `healthcheck:` to docker-compose services. |
| **Day 111** | Learn **Docker logs** — `docker logs`, `docker-compose logs -f`. Understand log drivers. Configure JSON file logging. |
| **Day 112** | **Month 4 Capstone:** Full system with Docker Compose — FastAPI + PostgreSQL + Redis + Celery Worker + Celery Beat + MinIO. Features: caching, background email, file uploads with async processing. Push to GitHub with full setup docs. |

---

## 🗓️ MONTH 5 — Cloud Hosting & Production Deployment
> **Theme:** Deploy your system to a real cloud server, enable HTTPS, and set up staging vs production.

---

### WEEK 17 — Cloud & Linux Server Basics

| Day | Target |
|-----|--------|
| **Day 113** | Create an AWS account (or GCP/Azure/DigitalOcean — pick one and stick with it). Understand: regions, availability zones, compute, storage, networking. |
| **Day 114** | Launch an EC2 instance (Ubuntu 22.04, t2.micro for free tier). SSH into it. Update packages. Install Docker and Docker Compose on the server. |
| **Day 115** | Learn Linux basics for servers — `systemctl`, `journalctl`, `htop`, `df -h`, `free -m`, `netstat`, `ufw` firewall. |
| **Day 116** | Configure **UFW firewall** — allow only ports 22 (SSH), 80 (HTTP), 443 (HTTPS). Block everything else. |
| **Day 117** | Learn **SSH key management** — create an SSH key pair, add public key to server, disable password auth. Set up SSH config on your local machine. |
| **Day 118** | Learn **environment separation** — create two `.env` files: `.env.staging` and `.env.production`. Understand why secrets differ between environments. |
| **Day 119** | **Weekend Mini Project:** Deploy your Month 4 Dockerized app to EC2. Access it via the public IP. Confirm all services are running. |

---

### WEEK 18 — Nginx & HTTPS

| Day | Target |
|-----|--------|
| **Day 120** | Install **Nginx** on your EC2 server. Configure it as a reverse proxy — forward `http://yourdomain.com` → `http://localhost:8000`. |
| **Day 121** | Register a domain name (Namecheap, GoDaddy, or use a free `.tk` domain). Point the DNS `A record` to your EC2 public IP. |
| **Day 122** | Install **Certbot**. Get a free SSL certificate from Let's Encrypt. Nginx auto-configured for HTTPS. Test HTTPS works. |
| **Day 123** | Configure **Nginx rate limiting** at the server level — limit connections per IP, limit request rate. Add as a first line of defense before FastAPI. |
| **Day 124** | Configure Nginx for **gzip compression** — compress API responses. Test with and without to see size difference. |
| **Day 125** | Configure **Nginx caching** for static files. Understand `proxy_cache` for API responses if needed. |
| **Day 126** | **Weekend Mini Project:** Full HTTPS production setup — domain + SSL + Nginx reverse proxy + your app running behind it. Share the URL with someone to test. |

---

### WEEK 19 — Production Best Practices

| Day | Target |
|-----|--------|
| **Day 127** | Set up **managed PostgreSQL** — AWS RDS free tier or a managed DB on DigitalOcean. Connect your app to the managed DB instead of a Docker DB. |
| **Day 128** | Set up **S3** (or DigitalOcean Spaces / Cloudflare R2) for file storage. Replace MinIO with real cloud object storage. Update your file upload code. |
| **Day 129** | Learn **secrets management** — use AWS Secrets Manager or Parameter Store. Never put real secrets in `.env` files in production repos. |
| **Day 130** | Configure **application logging** — structured JSON logs. Write logs to a file. Set up log rotation with `logrotate`. |
| **Day 131** | Set up **database backups** — write a script that dumps PostgreSQL to S3 every night. Test the restore process. |
| **Day 132** | Learn **zero-downtime deployments** — use `docker-compose up --no-deps --build api` to redeploy only the API container without downtime. |
| **Day 133** | **Weekend Mini Project:** Staging environment setup — launch a second EC2 or use the same server with port separation. Deploy to staging first, then production. |

---

### WEEK 20 — Production Hardening

| Day | Target |
|-----|--------|
| **Day 134** | Configure **Gunicorn** with Uvicorn workers — `gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app`. Understand worker processes. |
| **Day 135** | Set up a **bastion host** pattern — your DB should NOT be publicly accessible. Only the API (and your laptop via bastion) can reach it. |
| **Day 136** | Learn **AWS security groups** — configure so DB port 5432 is open only to the EC2 instance's security group, not the public internet. |
| **Day 137** | Configure **Redis AUTH** — set a Redis password. Update your app config. Never run Redis without auth in production. |
| **Day 138** | Learn **SSL for PostgreSQL** — configure your app to connect to RDS with `sslmode=require`. |
| **Day 139** | **Production Checklist:** Go through: HTTPS ✓, DB not public ✓, Redis password ✓, secrets not in code ✓, firewall configured ✓, backups working ✓. |
| **Day 140** | **Month 5 Capstone:** Your app is live on a real domain with HTTPS, managed DB, cloud file storage, proper secrets management, staging + production environments. Document everything in a `DEPLOYMENT.md`. |

---

## 🗓️ MONTH 6 — Scalability, Monitoring & CI/CD
> **Theme:** Automate deployments, monitor your system, and design for scale.

---

### WEEK 21 — CI/CD with GitHub Actions

| Day | Target |
|-----|--------|
| **Day 141** | Learn CI/CD concepts — Continuous Integration, Continuous Deployment. Understand pipelines, stages, and triggers. |
| **Day 142** | Create `.github/workflows/ci.yml`. On every push to `main`, run: `pytest` → `flake8` linting → `mypy` type checking. |
| **Day 143** | Add **Docker build** to CI — build your Docker image in the pipeline. Catch any build errors before deployment. |
| **Day 144** | Set up **GitHub Secrets** — store `EC2_HOST`, `EC2_USER`, `SSH_PRIVATE_KEY`, `DATABASE_URL`, etc. Use them in your workflow. |
| **Day 145** | Build the **CD pipeline** — on push to `main` (after CI passes): SSH into EC2, pull latest code, rebuild containers, restart services. |
| **Day 146** | Add **Docker Hub / GitHub Container Registry** — push your built image to a registry. Pull from registry on the server instead of building there. |
| **Day 147** | **Weekend Mini Project:** Full CI/CD working end-to-end — push code → tests run → image builds → deploys to server → site updates. Test it 5 times. |

---

### WEEK 22 — Logging & Monitoring

| Day | Target |
|-----|--------|
| **Day 148** | Set up **structured logging** with Python's `logging` module using JSON format. Log: request_id, user_id, method, path, status_code, duration_ms. |
| **Day 149** | Add a **request_id middleware** — generate a UUID for every request, attach it to logs and response headers. Useful for tracing issues. |
| **Day 150** | Install **Prometheus** and **Grafana** via docker-compose. Understand metrics: counters, gauges, histograms. |
| **Day 151** | Add `prometheus-fastapi-instrumentator` to your FastAPI app. Expose `/metrics` endpoint. See your API metrics in Prometheus. |
| **Day 152** | Build a **Grafana dashboard** — visualize: request rate, error rate (4xx/5xx), response time p50/p95/p99, active users. |
| **Day 153** | Set up **alerting** — Grafana alert if error rate > 5% for 5 minutes. Alert if response time p95 > 500ms. Send alert to email or Slack webhook. |
| **Day 154** | **Weekend Mini Project:** Full observability stack running — Prometheus scraping metrics, Grafana dashboard live, alerts configured. Generate some load with `wrk` and watch the graphs. |

---

### WEEK 23 — Scaling Strategies

| Day | Target |
|-----|--------|
| **Day 155** | Learn **horizontal vs vertical scaling**. When to scale up (bigger server) vs scale out (more servers). Understand the tradeoffs. |
| **Day 156** | Learn **load balancing** — configure Nginx as a load balancer across 2 FastAPI instances on the same server (different ports). |
| **Day 157** | Understand **stateless API design** — why your API must not store session state locally. Confirm your app is stateless (JWT, not server sessions). |
| **Day 158** | Learn **database read replicas** — understand read/write splitting. Route `SELECT` queries to replica, writes to primary. |
| **Day 159** | Learn **connection pooling at scale** — configure `PgBouncer` to sit between your app and PostgreSQL. Set pool size appropriately for your worker count. |
| **Day 160** | Learn **async FastAPI at scale** — understand `async def` vs `def` in FastAPI. Make all DB calls async with `asyncpg` + SQLAlchemy async mode. |
| **Day 161** | **Weekend Mini Project:** Load test your API with `locust` — simulate 100 concurrent users. Identify bottlenecks. Optimize the slowest endpoint. |

---

### WEEK 24 — Advanced Patterns & Final Project

| Day | Target |
|-----|--------|
| **Day 162** | Learn **database connection limits under load** — what happens when connections are exhausted. Configure proper pool sizes. |
| **Day 163** | Learn **Celery scaling** — run multiple Celery workers. Use `autoscale`. Route tasks to specific queues (high priority vs low priority). |
| **Day 164** | Learn **Redis Cluster vs Sentinel** — understand high availability for Redis. When each is appropriate. |
| **Day 165** | Learn **health check endpoints** — build `/health` (liveness) and `/health/ready` (readiness). Configure Nginx to use them. |
| **Day 166** | Learn **graceful shutdown** — handle `SIGTERM`. Finish in-flight requests before shutting down. Use `--graceful-timeout` in Gunicorn. |
| **Day 167** | Learn **API versioning strategy for breaking changes** — how to introduce `/v2` without breaking `/v1` clients. |
| **Day 168** | **Month 6 Capstone — Final System Review:** Run through the full stack. Fix any rough edges. Write a comprehensive `README.md` for your entire project. |

---

### FINAL DAYS — Polish & Portfolio

| Day | Target |
|-----|--------|
| **Day 169** | Write the system architecture document — diagrams, component descriptions, data flow, security model. |
| **Day 170** | Write a `RUNBOOK.md` — how to deploy, how to rollback, how to debug common issues, how to restore from backup. |
| **Day 171** | Performance baseline — document your system's performance: requests/sec, p95 latency, DB query times. |
| **Day 172** | Security review — check OWASP Top 10. Verify your system handles each. Document what's covered. |
| **Day 173** | Write 3 blog posts / LinkedIn posts about what you built. Explain one complex concept per post. |
| **Day 174** | Final GitHub cleanup — clean commit history, proper `.gitignore`, secrets check, working README, demo GIFs if possible. |
| **Day 175** | Mock interview prep — prepare to explain: your architecture decisions, why you chose each technology, how you'd scale to 10x traffic. |
| **Day 176** | Practice system design — on paper, design a URL shortener, a job queue, a rate limiter from scratch. |
| **Day 177** | Practice behavioral questions — talk through your 6-month project: challenges faced, decisions made, things you'd do differently. |
| **Day 178** | Apply to 5 Backend / AI Infrastructure roles. Tailor your resume to highlight the exact skills from this roadmap. |
| **Day 179** | Resume review + LinkedIn update — add all technologies, link your GitHub projects, write a strong summary. |
| **Day 180** | 🎉 **Day 180 — Graduation Day.** Review everything you've built. You now have the skills to be hired. Keep building. |

---

## 🏗️ Final System Architecture

```
                        ┌─────────────────────────────────────────────────────┐
                        │                   GitHub Actions CI/CD               │
                        │    push → test → build image → deploy to server      │
                        └─────────────────────────────────────────────────────┘
                                                  │
                                                  ▼
Internet ──────────► Nginx (Reverse Proxy + SSL + Rate Limiting + Load Balancer)
                                                  │
                          ┌───────────────────────┼───────────────────────┐
                          ▼                       ▼                       ▼
                   FastAPI Worker 1       FastAPI Worker 2       FastAPI Worker 3
                   (Gunicorn/Uvicorn)    (Gunicorn/Uvicorn)    (Gunicorn/Uvicorn)
                          │                       │                       │
                          └───────────────────────┼───────────────────────┘
                                                  │
                          ┌───────────────────────┼───────────────────────┐
                          │                       │                       │
                          ▼                       ▼                       ▼
                    PostgreSQL               Redis Cache           MinIO / S3
                    (Primary)               + Rate Limits         File Storage
                          │                 + Task Queue
                          ▼
                    PostgreSQL
                    (Read Replica)
                          │
                    ┌─────┴──────────┐
                    ▼                ▼
             Celery Worker    Celery Beat
             (Async Jobs)     (Scheduled)
                          │
                    ┌─────┴──────────┐
                    ▼                ▼
               Prometheus         Grafana
               (Metrics)         (Dashboard +
                                   Alerts)
```

---

## 📦 Full Tech Stack Summary

| Category | Technologies |
|----------|-------------|
| **Language** | Python 3.11+ |
| **API Framework** | FastAPI + Uvicorn + Gunicorn |
| **Data Validation** | Pydantic v2 |
| **ORM** | SQLAlchemy 2.0 (async) |
| **Migrations** | Alembic |
| **Database** | PostgreSQL 15 |
| **Caching** | Redis 7 |
| **Task Queue** | Celery + Redis Broker |
| **Auth** | JWT (python-jose) + bcrypt (passlib) |
| **File Storage** | MinIO (local) → AWS S3 / Cloudflare R2 (prod) |
| **Containerization** | Docker + Docker Compose |
| **Web Server** | Nginx |
| **SSL** | Let's Encrypt + Certbot |
| **Cloud** | AWS EC2 + RDS + S3 (or DigitalOcean) |
| **CI/CD** | GitHub Actions |
| **Monitoring** | Prometheus + Grafana |
| **Testing** | pytest + pytest-cov + TestClient |
| **Load Testing** | Locust + wrk |

---

## 📁 Final Project Folder Structure

```
my-backend/
├── app/
│   ├── __init__.py
│   ├── main.py                  # FastAPI app entry point
│   ├── config.py                # Environment settings (Pydantic BaseSettings)
│   ├── database.py              # DB engine, session, Base
│   │
│   ├── models/                  # SQLAlchemy ORM models
│   │   ├── user.py
│   │   ├── post.py
│   │   └── ...
│   │
│   ├── schemas/                 # Pydantic request/response schemas
│   │   ├── user.py
│   │   ├── post.py
│   │   └── ...
│   │
│   ├── routers/                 # FastAPI route handlers
│   │   ├── v1/
│   │   │   ├── auth.py
│   │   │   ├── users.py
│   │   │   ├── posts.py
│   │   │   └── ...
│   │   └── health.py
│   │
│   ├── services/                # Business logic layer
│   │   ├── auth_service.py
│   │   ├── user_service.py
│   │   ├── post_service.py
│   │   └── ...
│   │
│   ├── tasks/                   # Celery tasks
│   │   ├── email_tasks.py
│   │   ├── file_tasks.py
│   │   └── ...
│   │
│   ├── middleware/              # Custom middleware
│   │   ├── logging.py
│   │   └── request_id.py
│   │
│   ├── core/                   # Core utilities
│   │   ├── security.py         # JWT, hashing
│   │   ├── cache.py            # Redis cache helpers
│   │   └── exceptions.py       # Custom exceptions
│   │
│   └── dependencies.py         # Shared FastAPI dependencies
│
├── alembic/                    # Database migrations
│   ├── versions/
│   ├── env.py
│   └── alembic.ini
│
├── tests/                      # Test suite
│   ├── conftest.py
│   ├── test_auth.py
│   ├── test_users.py
│   └── ...
│
├── .github/
│   └── workflows/
│       ├── ci.yml              # Run tests + lint
│       └── cd.yml              # Deploy to production
│
├── docker/
│   ├── Dockerfile
│   ├── Dockerfile.celery
│   └── nginx.conf
│
├── docker-compose.yml          # Local development
├── docker-compose.prod.yml     # Production overrides
├── .env.example                # Template (never commit .env)
├── requirements.txt
├── README.md
├── DEPLOYMENT.md
└── RUNBOOK.md
```

---

## 🎯 Monthly Milestone Checklist

### ✅ End of Month 1 — Database
- [ ] PostgreSQL installed and comfortable with `psql`
- [ ] Complex queries with JOINs, aggregates, subqueries
- [ ] Indexes understood and implemented
- [ ] Transactions written and tested
- [ ] SQLAlchemy models created
- [ ] Alembic migrations working
- [ ] E-Commerce schema on GitHub

### ✅ End of Month 2 — API
- [ ] FastAPI CRUD endpoints working
- [ ] Pydantic validation on all inputs/outputs
- [ ] Pagination and filtering implemented
- [ ] Global error handling
- [ ] API versioning (`/v1`)
- [ ] Service layer separation
- [ ] 80%+ test coverage
- [ ] Blog API on GitHub

### ✅ End of Month 3 — Auth & Security
- [ ] JWT auth working (login, protected routes)
- [ ] Refresh tokens implemented
- [ ] RBAC (user/moderator/admin) working
- [ ] Rate limiting with Redis
- [ ] API key system working
- [ ] Email verification + password reset
- [ ] Token blacklisting on logout
- [ ] Secure Auth System on GitHub

### ✅ End of Month 4 — Docker & Background Tasks
- [ ] App fully Dockerized
- [ ] Docker Compose with all services
- [ ] Redis caching on key endpoints
- [ ] Cache invalidation working
- [ ] Celery worker running background jobs
- [ ] Celery Beat for scheduled tasks
- [ ] MinIO file uploads working
- [ ] Full system on GitHub

### ✅ End of Month 5 — Cloud & Production
- [ ] App deployed on EC2 (or equivalent)
- [ ] Domain name configured
- [ ] HTTPS with Let's Encrypt
- [ ] Nginx reverse proxy configured
- [ ] Managed PostgreSQL (RDS or equivalent)
- [ ] S3 for file storage
- [ ] Staging + production environments
- [ ] `DEPLOYMENT.md` written

### ✅ End of Month 6 — Scale & Monitoring
- [ ] CI/CD pipeline working end-to-end
- [ ] Prometheus metrics exposed
- [ ] Grafana dashboard live
- [ ] Alerts configured
- [ ] Load balancing configured
- [ ] Load test run and documented
- [ ] `RUNBOOK.md` written
- [ ] Portfolio-ready GitHub README

---

## 🔥 Key Principles to Never Forget

> **1. Database is the foundation.** A poorly designed schema is impossible to fix at scale. Design it right from day one.

> **2. Errors are first-class citizens.** Every endpoint should return consistent, meaningful error responses. Never return a raw exception.

> **3. Never trust user input.** Validate everything at the API boundary with Pydantic. Parameterize all DB queries.

> **4. Secrets never go in code.** Use environment variables. Use secrets managers in production. Rotate secrets regularly.

> **5. Log everything meaningful.** If you can't see what your system is doing, you can't fix it when it breaks. And it will break.

> **6. Design for failure.** DB goes down, Redis is unavailable, a Celery worker crashes. Handle each case gracefully.

> **7. Automate deployments.** Manual deployments introduce human error. Automate everything and deploy 10x a day if needed.

> **8. Test before you ship.** A test suite is not optional. It is the confidence that lets you deploy without fear.

---

## 📚 Recommended Resources

| Topic | Resource |
|-------|----------|
| FastAPI | [fastapi.tiangolo.com](https://fastapi.tiangolo.com) — official docs are exceptional |
| PostgreSQL | [pgexercises.com](https://pgexercises.com) — hands-on SQL practice |
| SQLAlchemy | [docs.sqlalchemy.org](https://docs.sqlalchemy.org) — read the 2.0 tutorial |
| Docker | [docs.docker.com/get-started](https://docs.docker.com/get-started) |
| Redis | [redis.io/docs](https://redis.io/docs) |
| Celery | [docs.celeryq.dev](https://docs.celeryq.dev) |
| Nginx | [nginx.org/en/docs](https://nginx.org/en/docs) |
| GitHub Actions | [docs.github.com/actions](https://docs.github.com/en/actions) |
| System Design | "Designing Data-Intensive Applications" by Martin Kleppmann |
| AWS | [aws.amazon.com/free](https://aws.amazon.com/free) — start with free tier |

---

## 💼 What This Roadmap Gets You

After completing this roadmap you will be qualified for:

- **Backend Engineer** — Mid to Senior level
- **AI Infrastructure Engineer** — Deploy and maintain AI systems
- **Platform Engineer** — Build internal developer platforms
- **DevOps / SRE** — With additional cloud certifications
- **Full-Stack Engineer** (Backend side) — Add frontend on top

**Salary range (2025–2026):** $90K–$180K depending on location and level.

---

*Built for engineers who want real skills, not tutorial certificates.*
*Push code every day. Break things. Fix them. Ship.*

---

**⭐ Star this repo if it helped you. Fork it. Track your progress. Share it with someone who needs it.**
