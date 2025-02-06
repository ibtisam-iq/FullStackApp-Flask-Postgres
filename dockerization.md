# Dockerizing a Full-Stack Flask-PostgreSQL Application

## Application Structure

This is a 2-tier architecture because the frontend (HTML, CSS) is served directly by the Flask backend, and the backend communicates with the PostgreSQL database. The client interacts with the backend, which in turn interacts with the database, making it a tightly coupled system.

## Project Structure

```plaintext
.
├── app
│   ├── __init__.py
│   ├── models.py
│   ├── routes.py
│   ├── static
│   │   └── css
│   │       └── style.css
│   └── templates
│       ├── create.html
│       ├── delete.html
│       ├── edit.html
│       ├── index.html
│       ├── layout.html
│       ├── update.html
│       └── view.html
├── config.py
├── .gitignore
├── requirements.txt
├── run.py
├── Dockerfile
├── Dockerfile.frontend
├── docker-compose.yml
└── tests
    ├── __init__.py
    └── test_app.py
```
## Database Integration

The database connection string is set as `postgresql://ibtisam:ibtisam@db/loveyou_ibtisam` in `config.py`.

---
## Multi-Stage Dockerfile

### Without Virtual Environment

```dockerfile
# filepath: Dockerfile
# Stage 1: Build the application
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .

# Stage 2: Create the final image
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /app /app
ENV FLASK_APP=run.py
ENV FLASK_ENV=development
ENV DATABASE_URL=postgresql://ibtisam:ibtisam@db/loveyou_ibtisam
EXPOSE 5000
CMD ["python", "run.py"]
```

### With Virtual Environment

```dockerfile
# filepath: Dockerfile
# Stage 1: Build the application
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN python -m venv venv && . venv/bin/activate && pip install --no-cache-dir -r requirements.txt
COPY . .

# Stage 2: Create the final image
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /app /app
ENV PATH="/app/venv/bin:$PATH"
ENV FLASK_APP=run.py
ENV FLASK_ENV=development
ENV DATABASE_URL=postgresql://ibtisam:ibtisam@db/loveyou_ibtisam
EXPOSE 5000
CMD ["python", "run.py"]
```

## Docker Compose File

```yaml
# filepath: docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      - FLASK_APP=run.py
      - FLASK_ENV=development
      - DATABASE_URL=postgresql://ibtisam:ibtisam@db/loveyou_ibtisam
    depends_on:
      - db
      - frontend

  db:
    image: postgres:13
    environment:
      POSTGRES_USER: ibtisam
      POSTGRES_PASSWORD: ibtisam
      POSTGRES_DB: loveyou_ibtisam
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## Commands to Run

1. **Build the containers:**
   ```bash
   docker-compose build
   ```

2. **Start the services:**
   ```bash
   docker-compose up
   ```

3. **Run tests within the Flask container:**
   ```bash
   docker-compose exec web pytest tests
   ```
## Reference

You can find in-depth information [here](Dockerization.md).


