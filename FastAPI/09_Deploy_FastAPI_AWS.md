# 📘 Lecture Notes: Deploying a FastAPI + Docker ML Model on AWS EC2

---

## 1. **Context Recap of the Playlist**

* **Project Goal**: Build and deploy an ML model API for **insurance premium prediction**.

* **Steps covered previously**:

  1. Built ML model.
  2. Created **FastAPI backend** with `/predict` endpoint.
  3. Improved API with **best practices**.
  4. **Dockerized** the API → pushed Docker image to **Docker Hub**.

* **Today’s goal**:

  * Deploy Dockerized FastAPI API to **AWS EC2**.
  * Configure API access via **public IP**.
  * Integrate with **Streamlit frontend**.

---

## 2. **Pre-Requisites**

* AWS account (requires debit/credit card for setup, but free-tier available).
* Basic knowledge of:

  * Cloud fundamentals
  * AWS services (EC2 in particular)
  * Docker commands

---

## 3. **Introduction to AWS EC2**

* **EC2 (Elastic Compute Cloud)** = rented virtual machines on AWS.
* You can scale up/down the number of EC2 instances as needed.
* For this tutorial: deploy on **1 EC2 instance** (free-tier: `t2.micro` with 1GB RAM).

---

## 4. **Step 1: Create an EC2 Instance**

1. Log in to **AWS Console**.
2. Search for **EC2** → go to **Instances**.
3. Click **Launch Instances**.

   * **Name**: e.g., `my-fastapi-server`
   * **AMI (OS Image)**: Ubuntu
   * **Instance Type**: `t2.micro` (free-tier)
   * **Key Pair**:

     * Needed for SSH access.
     * Download `.pem` file if creating new.
   * **Network Settings**:

     * Allow **SSH (22)** and **HTTP/HTTPS** traffic.
     * Ensure "Anywhere" access is selected.
   * **Storage**: Default 8 GB is fine.
4. Click **Launch Instance**.
5. Verify instance status: should be **Running**.

---

## 5. **Step 2: Connect to EC2 Instance**

* Two ways:

  1. Using `.pem` key with SSH (via terminal or PuTTY).
  2. Using AWS Console → **Connect** button (browser-based terminal).
* After connecting, you’ll have command-line access to the VM.

---

## 6. **Step 3: Install & Setup Docker on EC2**

Run the following commands in the EC2 terminal:

```bash
# Update system packages
sudo apt update -y

# Install Docker
sudo apt install docker.io -y

# Start Docker service
sudo systemctl start docker

# Enable Docker to auto-start on reboot
sudo systemctl enable docker

# Verify Docker installation
sudo docker --version
sudo docker ps
```

✅ At this stage, Docker is running and ready.

---

## 7. **Step 4: Pull and Run the Docker Image**

1. **Grant permissions** to EC2 for Docker Hub access:

   ```bash
   sudo usermod -aG docker ubuntu
   exit  # logout and reconnect
   ```

2. **Pull Docker image** from Docker Hub:

   ```bash
   sudo docker pull <your-dockerhub-username>/<image-name>
   ```

3. **Run Docker container**:

   ```bash
   sudo docker run -d -p 8000:8000 <your-dockerhub-username>/<image-name>
   ```

   * `-d` → run in background
   * `-p 8000:8000` → map container’s port to EC2’s public port

4. Check if container is running:

   ```bash
   sudo docker ps
   ```

---

## 8. **Step 5: Configure Security Groups**

* By default, port `8000` is blocked.
* Go to **EC2 → Security Groups → Inbound Rules**.
* Add a rule:

  * **Type**: Custom TCP
  * **Port**: 8000
  * **Source**: Anywhere (0.0.0.0/0)
* Save rules.

---

## 9. **Step 6: Access Your API**

* Copy **Public IPv4 Address** from EC2 instance details.
* Visit in browser:

  ```
  http://<EC2-PUBLIC-IP>:8000
  ```
* Endpoints available:

  * `/` → Home
  * `/health` → Health check
  * `/docs` → Swagger API docs (interactive testing)

---

## 10. **Common Issues**

* ❌ `HTTPS not working` → need SSL certificate (not covered in beginner setup).
* ❌ API not accessible → Check **Security Group rules** (port 8000 must be open).
* ❌ Container stops after reboot → Ensure Docker is enabled.

---

## 11. **Step 7: Connect FastAPI API with Streamlit Frontend**

* Existing **Streamlit app** used for prediction UI.
* Only change required:

  * Update API URL from `localhost:8000` → `<EC2-PUBLIC-IP>:8000`.

**Example:**

```python
import requests
import streamlit as st

API_URL = "http://<EC2-PUBLIC-IP>:8000/predict"

st.title("Insurance Premium Prediction")

age = st.number_input("Age", 18, 100)
bmi = st.number_input("BMI", 10.0, 50.0)
children = st.number_input("Children", 0, 10)
smoker = st.selectbox("Smoker", ["yes", "no"])
region = st.selectbox("Region", ["northwest", "northeast", "southeast", "southwest"])

if st.button("Predict"):
    payload = {
        "age": age,
        "bmi": bmi,
        "children": children,
        "smoker": smoker,
        "region": region
    }
    response = requests.post(API_URL, json=payload)
    st.write("Predicted Premium:", response.json()["prediction"])
```

* Run Streamlit app locally:

  ```bash
  streamlit run app.py
  ```
* App communicates with deployed FastAPI API on AWS.

---

## 12. **Summary of Workflow**

1. **Built ML model** → Exposed via **FastAPI**.
2. **Improved API** with best practices.
3. **Dockerized** API → pushed to **Docker Hub**.
4. **Created EC2 instance** on AWS.
5. **Installed Docker** on EC2.
6. **Pulled and ran Docker container**.
7. **Configured Security Groups** → exposed port `8000`.
8. **Accessed API via public IP**.
9. **Updated Streamlit frontend** → API now works over the cloud.

---

## 13. **Next Steps (Beyond Beginner Level)**

* Add **SSL certificates** for HTTPS (e.g., with Nginx + Let’s Encrypt).
* Use **Elastic Beanstalk / ECS / EKS** for scalable deployments.
* Implement **CI/CD pipelines** for automated deployment.
* Add monitoring/logging tools.

---

✅ By following these steps, you’ve completed a full cycle: **ML model → API → Docker → Cloud Deployment → Frontend Integration**.
