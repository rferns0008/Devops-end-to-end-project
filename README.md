# DevOps Setup Guide
> End-to-end setup for Java + Maven + Docker + Jenkins + Minikube on an AWS EC2 Ubuntu instance.

---

## 1. SSH into the EC2 Instance

Connect to your remote server using the provided `.pem` key file.

```bash
chmod 400 Microdegree-test.pem
ssh -i Microdegree-test.pem ubuntu@54.91.1.235
```

> `chmod 400` restricts the key file to read-only for the owner, which is required by SSH for security.

---

## 2. Install Java (OpenJDK 21)

Jenkins and Maven require Java. Install OpenJDK 21 and verify the installation.

```bash
sudo apt update
sudo apt install openjdk-21-jdk -y
java -version
```

---

## 3. Install Maven

Maven is used to build the Java application.

```bash
sudo apt install maven -y
mvn -version
```

---

## 4. Install Docker

Docker is needed to build and run containerized images, and also to drive Minikube.

```bash
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu
```

> After running `usermod`, **log out and log back in** so the group change takes effect. Verify with:
> ```bash
> docker --version
> ```

---

## 5. Install Jenkins

Jenkins is downloaded as a standalone WAR file and run directly using Java.

```bash
wget https://get.jenkins.io/war-stable/latest/jenkins.war
java -jar jenkins.war --httpPort=8080
```

> Jenkins will be accessible at `http://54.91.1.235:8080`. On first launch, it will display an initial admin password in the terminal output.

---

## 6. Install kubectl & Minikube

`kubectl` is the CLI for interacting with Kubernetes. Minikube runs a local single-node cluster.

```bash
# Install kubectl
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start Minikube using Docker as the driver
minikube start --driver=docker --memory=2200mb
```

---

## 7. Build the Application

Use Maven to compile and package the Java application into a JAR/WAR file.

```bash
mvn clean package
```

---

## 8. Docker – Build, Run & Push

Build the Docker image, test it locally, then push it to Docker Hub.

```bash
# Build the image
docker build -t pranavpk0107/webapp:latest .

# Run the container locally to test
docker run -d --name webapp pranavpk0107/webapp:latest

# Login to Docker Hub and push the image
docker login
docker push pranavpk0107/webapp:latest
```

---

## 9. Deploy to Kubernetes

Apply your Kubernetes manifests and expose the application via port-forwarding.

```bash
# Apply all manifests in the k8s/ directory
kubectl apply -f k8s/

# Forward the service port to access it from outside
kubectl port-forward service/webapp-service 9090:8080 --address 0.0.0.0
```

> The app will be accessible at `http://54.91.1.235:9090`.

---

## 10. Configure GitHub Webhook for Jenkins

Automate builds on every push by connecting GitHub to Jenkins via a webhook.

1. In your Jenkins job config, enable **"GitHub hook trigger for GITScm polling"**.
2. In your GitHub repository, go to **Settings → Webhooks → Add webhook**.
3. Set the Payload URL to:
   ```
   http://54.91.1.235:8080/github-webhook/
   ```
4. Set **Content type** to `application/json`.
5. Select **"Just the push event"** and save.

> Every push to the repository will now automatically trigger a Jenkins build.

---

## 11. Cleanup – Delete Kubernetes Resources

Remove deployed resources from the cluster when no longer needed.

```bash
kubectl delete deployment webapp-deployment
kubectl delete service webapp-service

# Verify all resources are gone
kubectl get all
```
