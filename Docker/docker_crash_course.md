# 🚀 Lecture Notes on Docker (Beginner to Intermediate for Data Science & MLOps)

---

## 1. Introduction

* **Topic:** Docker (a *universal* software tool).
* **Importance:**

  * Used in *every software profile*: developers, data scientists, MLOps engineers, DevOps engineers, etc.
  * *High demand* in jobs and interviews.
* **Lecture plan:**

  1. **Theory (\~40 mins)**: Fundamentals, components, architecture.
  2. **Practical (\~50 mins)**: Build, push, pull, and run Docker images for an ML project.

---

## 2. What is Docker?

* **Definition (Wikipedia):**
  Docker is a platform to help developers **build, share, and run containerized applications**.

* **Analogy (Maggi Noodles):**

  * Maggi provides **standard masala in a packet**. Wherever you cook it, taste remains same.
  * If Maggi only gave a recipe book, taste would vary due to local ingredient quality.
  * ⇒ Standardization ensures *consistency everywhere*.
  * **In software:** Docker standardizes environment so that software runs identically across machines.

---

## 3. Why Do We Need Docker?

### 3.1 Consistency Across Environments

* Problem:

  * Code works on developer’s machine but breaks on tester’s or production due to OS/library/version differences.
* Example:

  * Developer uses scikit-learn v0.22, tester has v1.2 → code breaks.
* **Solution:** Docker “locks” code + dependencies into a single package → same behavior everywhere.

---

### 3.2 Isolation

* One server may host **multiple apps/websites**.
* Without isolation:

  * Hack in one app may affect others.
  * One app may consume all CPU/RAM, starving others.
* **Solution:** Docker containers provide isolated environments (lighter than Virtual Machines).

---

### 3.3 Scalability

* Traffic fluctuates.
* Docker allows **quickly running multiple containers** of same app on different machines.
* When traffic decreases, containers can be shut down.
* Very useful for **load balancing and elastic scaling**.

---

## 4. Key Docker Concepts

### 4.1 Docker Engine

* **Core component** of Docker platform.
* Responsible for creating, running, and managing containers.

**Components:**

1. **Docker Daemon (dockerd):** Runs in background, manages images, containers, volumes, networks.
2. **Docker CLI:** Command-line tool (`docker run`, `docker build`, etc.) for user interaction.
3. **REST API:** Communication layer between CLI and daemon (can also enable programmatic automation).

---

### 4.2 Docker Image

* **Definition:**
  Lightweight, standalone, executable package containing everything needed to run software:

  * Code, runtime, libraries, environment variables, configs.

* **Lifecycle:**

  1. **Create** → `docker build -t image_name .`
  2. **Store** → in registry (Docker Hub, private, or cloud ECR/GCR/ACR).
  3. **Distribute** → others pull it.
  4. **Execute** → `docker run image_name` to create a container.

---

### 4.3 Dockerfile

* **Definition:**
  A text file with **instructions** to build an image.

**Common Instructions:**

```dockerfile
FROM python:3.9              # Base image (OS + Python runtime)
WORKDIR /app                 # Working directory inside container
COPY . /app                  # Copy project files
RUN pip install -r requirements.txt   # Install dependencies
EXPOSE 5000                  # Open port for Flask app
CMD ["python", "app.py"]     # Command to run app
```

* Each instruction creates a **layer** in the final image.

---

### 4.4 Docker Container

* **Definition:**
  A running instance of an image.
* **Analogy:**

  * Image = DVD of a movie.
  * Container = Movie playing on TV/projector.

---

### 4.5 Docker Registry

* **Definition:**
  Service to **store and distribute images**.

* **Types:**

  * Public (Docker Hub).
  * Private (organization-specific).
  * Cloud (AWS ECR, GCP GCR, Azure ACR).

* **Key components:**

  * Repository (collection of related images).
  * Tags (versions, e.g. `v1`, `v2`, `latest`).

---

## 5. How Docker Works (Workflow)

**Step-by-step:**

1. Developer writes code + Dockerfile.
2. Build image: `docker build -t username/app_name .`
3. Test locally: `docker run -p 8080:5000 username/app_name`.
4. Push to registry: `docker push username/app_name`.
5. Tester/teammate pulls: `docker pull username/app_name`.
6. Run container on their machine: `docker run ...`.

---

## 6. Use Cases of Docker

* **Microservices architecture** (separate containers for each service).
* **CI/CD pipelines** (consistent environment for build → test → deploy).
* **Cloud migration** (easy portability between providers).
* **Scalable web applications** (spin up/down containers based on traffic).
* **Testing & QA** (share image instead of code).
* **Machine Learning/AI** (reproducible environments for training and deployment).
* **API development** (consistent backend runtime).

---

## 7. Tools for Practical Work

* **Docker Desktop**:

  * Provides Docker Engine + GUI for managing images/containers.
  * GUI shows running containers, stored images, volumes.
  * CLI integrated for running commands.

* **Docker Hub**:

  * Website for searching/uploading/pulling images.
  * Requires login (`docker login`).

---

## 8. Practical Examples

### 8.1 Hello World (sanity check)

```bash
docker pull hello-world
docker run hello-world
```

Expected output: `"Hello from Docker!"`

---

### 8.2 Flask App Example

**app.py:**

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/')
def home():
    num = int(request.args.get("num", 2))
    return "<br>".join([f"{num} x {i} = {num*i}" for i in range(1, 11)])

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**requirements.txt:**

```
flask
```

**Dockerfile:**

```dockerfile
FROM python:3.9
WORKDIR /app
COPY . /app
RUN pip install -r requirements.txt
EXPOSE 5000
CMD ["python", "app.py"]
```

**Build & Run:**

```bash
docker build -t username/flask-table .
docker run -p 8888:5000 username/flask-table
```

Visit: `http://localhost:8888`

---

### 8.3 ML Project Example (Laptop Price Predictor)

**Dockerfile:**

```dockerfile
FROM python:3.7
WORKDIR /app
COPY . /app
RUN pip install -r requirements.txt
EXPOSE 8501
CMD ["streamlit", "run", "app.py"]
```

**Build:**

```bash
docker build -t username/laptop-predictor .
```

**Run locally:**

```bash
docker run -p 8501:8501 username/laptop-predictor
```

**Push to Docker Hub:**

```bash
docker login
docker push username/laptop-predictor
```

**Pull & Run on another machine:**

```bash
docker pull username/laptop-predictor
docker run -p 8501:8501 username/laptop-predictor
```

---

## 9. Important Concepts to Remember

* **Port Mapping:**
  `-p host_port:container_port` (e.g., `-p 8888:5000`).

  * Container runs on internal port → mapped to host port.

* **Version Compatibility:**
  Sometimes Python/library versions must match the project’s original environment.

* **Lightweight vs Heavy Images:**

  * Using `python:3.9` base image adds OS → bigger (\~1GB).
  * Can use **slim or alpine images** to reduce size.

---

## 10. Summary

* **Docker = Consistency + Isolation + Scalability**.
* Workflow = *Write Dockerfile → Build image → Push → Pull → Run container*.
* Essential for: microservices, CI/CD, ML model deployment, reproducibility.
* Mastering Docker is crucial for Data Scientists and MLOps engineers.

---

✅ If you **revise from these notes + practice commands**, you’ll have a full understanding of Docker at beginner-to-intermediate level.
