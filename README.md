# Hands-On L3: Installing Docker & Building a Multi-Container Microservice

This hands-on will guide you through installing Docker Desktop, deploying a Python Flask web application with a Redis cache, and running a PostgreSQL database. You’ll practice container networking, multi-container orchestration with Docker Compose, and submit your results via GitHub.

## What You’ll Do
- Install and verify Docker Desktop
- Run a PostgreSQL container
- Set up Flask app and Redis cache files
- Build and connect services with Docker Compose
- Test in browser and capture screenshots of output and Docker Desktop showing the containers
- Push project to GitHub
- Create an Issue for errors faced during handson (intentional errors included)

---

## 1. Install Docker Desktop

### Windows
1. Go to Docker Desktop for Windows
2. Download Docker Desktop Installer.exe
3. Run it with default settings
4. Start Docker Desktop from the Start Menu
5. If prompted, enable WSL 2 and restart

### macOS
1. Go to Docker Desktop for Mac
2. Download the installer for your chip type (Apple Silicon or Intel)
3. Open the .dmg file and drag Docker.app into Applications
4. Open Docker.app and follow security prompts

### Verify Installation
Open Command Prompt (Windows) or Terminal (Mac) and run:

```sh
docker -v
```

---

## 2. PostgreSQL Setup with Docker

```sh
docker pull postgres
docker run -d -p 5432:5432 --name postgres1 -e POSTGRES_PASSWORD=pass12345 postgres
docker exec -it postgres1 bash
psql -d postgres -U postgres
```

---

## 3. Build a Python Web App with Docker Compose

### a. Define dependencies in `requirements.txt`:
```
flask
redis
```

### b. Application in `app.py`:
```python
import time
import redis
from flask import Flask
app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)
def get_hit_count():
	retries = 5
	while True:
		try:
			return cache.incr('hits')
		except redis.exceptions.ConnectionError as exc:
			if retries == 0:
				raise exc
			retries -= 1
			time.sleep(0.5)
@app.route('/')
def hello():
	count = get_hit_count()
	return f'Hello World! I have been seen {count} times.\n'
```

### c. `Dockerfile`:
```Dockerfile
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

### d. `compose.yaml`:
```yaml
version: "3.9"
services:
  web:
	build: .
	ports:
	  - "8000:5000"
	depends_on:
	  - redis
  redis:
	image: "redis:alpine"
```

### e. Build and run the application:
```sh
docker compose up
```

### f. Open the application in a browser:
http://localhost:8000

---

## Submission
1. Upload all files to GitHub (`app.py`, `Dockerfile`, `compose.yaml`, `requirements.txt`, `README.md`, and a document with screenshots)
2. **README.md**: This file should contain the execution steps and what you learned in this handson, with appropriate formatting.
3. Log the errors faced in handson (intentional errors) under the Issues tab in GitHub.
4. Upload screenshots of:
	- Docker Desktop app showing the containers
	- Output obtained after the handson
5. Submit the link of the GitHub repository in Canvas.

---

## What I Learned

- How to install and verify Docker Desktop
- Running and connecting multiple containers using Docker Compose
- Container networking between Flask, Redis, and PostgreSQL
- Debugging and logging issues using GitHub Issues
- Submitting results and documentation via GitHub