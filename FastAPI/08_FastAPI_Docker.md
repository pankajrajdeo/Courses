# 🚀 Lecture Notes: Dockerizing a FastAPI ML Prediction API and Uploading to Docker Hub

---

## 1. 📌 Context and Recap

* **Previous Videos Covered:**

  * Fundamentals of **FastAPI**.
  * Built a **machine learning model** (Insurance Premium Prediction).
  * Exposed the ML model via a **prediction API endpoint** (`/predict`).
  * Improved the simple API by applying **best practices** to make it more industry-grade.

* **Current Lecture Goals:**

  1. **Dockerize** the improved FastAPI application.
  2. **Push the Dockerized image** to Docker Hub so it can be shared and run anywhere.

---

## 2. 📦 Why Docker?

* **Problem:** Deploying APIs across different machines can cause compatibility issues (e.g., Python versions, missing libraries, OS dependencies).
* **Solution: Docker**

  * Packages application + dependencies into a **single image**.
  * Runs consistently on **any machine** with Docker installed.
  * Makes sharing easy: just push the image to **Docker Hub**.
* **Industry use case:** Testers, teammates, or cloud platforms (AWS, GCP, Azure) can run the same image without setup issues.

---

## 3. ⚙️ Prerequisites

1. **Docker Desktop** installed on your machine.

   * Download from: [https://www.docker.com](https://www.docker.com).
   * Install normally (Next → Next → Finish).
2. **Docker Hub account** created at [https://hub.docker.com](https://hub.docker.com).

   * Used to host and share images.

---

## 4. 📝 Creating a Dockerfile

A **Dockerfile** is an instruction manual for building a Docker image.

### Steps inside Dockerfile:

1. **Choose base image** → A Python image with the required version.
2. **Set working directory** inside container (e.g., `/app`).
3. **Copy `requirements.txt`** and install dependencies.
4. **Copy application code** (FastAPI files, ML model files).
5. **Expose port** → FastAPI typically runs on port `8000`.
6. **Run application** using `uvicorn`.

### Example `Dockerfile`

```dockerfile
# 1. Use an official Python base image
FROM python:3.9

# 2. Set working directory
WORKDIR /app

# 3. Copy dependency list
COPY requirements.txt .

# 4. Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# 5. Copy application code
COPY . .

# 6. Expose port 8000 (FastAPI default)
EXPOSE 8000

# 7. Run the FastAPI app with Uvicorn
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## 5. 🛠️ Building the Docker Image

1. Open terminal inside project directory.

2. Run build command:

   ```bash
   docker build -t <dockerhub-username>/<image-name>:<tag> .
   ```

   * Example:

     ```bash
     docker build -t tws24/insurance-api:latest .
     ```
   * `tws24` → Docker Hub username.
   * `insurance-api` → Repository (image) name.
   * `latest` → Default tag.

3. Verify image built successfully:

   ```bash
   docker images
   ```

   OR check inside **Docker Desktop → Images** tab.

---

## 6. ▶️ Running the Docker Container

Run image locally as a container:

```bash
docker run -d -p 8000:8000 tws24/insurance-api:latest
```

* `-d` → detached mode.
* `-p 8000:8000` → maps container’s port `8000` to host machine’s port `8000`.
* Visit API in browser:

  * Home: `http://localhost:8000/`
  * Docs: `http://localhost:8000/docs`

---

## 7. ☁️ Uploading Image to Docker Hub

### Step 1: Login

```bash
docker login
```

* Enter **username** + **password** for Docker Hub.

### Step 2: Push image

```bash
docker push tws24/insurance-api:latest
```

* Uploads layer by layer to Docker Hub.
* Check progress in terminal.
* After completion, image appears on Docker Hub account.

---

## 8. 🔄 Pulling & Testing as a Different User (Tester’s View)

Imagine you are a tester in the same organization:

### Step 1: Pull the image

```bash
docker pull tws24/insurance-api:latest
```

### Step 2: Run the container

```bash
docker run -d -p 8000:8000 tws24/insurance-api:latest
```

### Step 3: Test endpoints

Open browser:

* **Docs**: `http://localhost:8000/docs`
* **Prediction test** (`/predict`):
  Example request body:

  ```json
  {
    "age": 31,
    "weight": 91,
    "height": 1.72,
    "income": 10,
    "smoker": true,
    "city": "Gurgaon",
    "occupation": "Retired"
  }
  ```
* API responds with prediction: **Low / Medium / High Insurance Premium**.

---

## 9. 🧠 Key Concepts Recap

* **Docker Image**: Packaged application (blueprint).
* **Container**: Running instance of an image.
* **Docker Hub**: Online registry to share Docker images.
* **Why Docker?**

  * Consistency across machines.
  * Easy distribution (share via Docker Hub).
  * Works seamlessly with cloud deployment.

---

## 10. ✅ What’s Next?

* In the **next lecture**: Deploy this Docker image to **AWS** so it becomes publicly accessible on the internet.
* This will make the FastAPI prediction API available to anyone with the link.

---

# 📖 Final Summary

In this lecture, we:

1. Built a **Docker image** of the FastAPI ML prediction API using a **Dockerfile**.
2. **Ran and tested** it locally with Docker.
3. **Pushed the image to Docker Hub** for sharing.
4. Verified as a **tester** by pulling and running it from Docker Hub.

This workflow ensures the API can be run **anywhere** without dependency issues, setting the stage for **cloud deployment**.
