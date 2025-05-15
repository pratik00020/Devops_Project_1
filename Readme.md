# 🧱 Maven Web App Deployment with Jenkins + Docker + Tomcat

This project demonstrates a complete CI/CD pipeline for deploying a Java-based Maven web application using:

- **Git** for source control
- **Maven** for building the `.war` file
- **Jenkins** for continuous integration and deployment
- **Docker** for containerization
- **Tomcat (in Docker)** as the deployment server

---

## 📁 Project Structure

```
.
├── Dockerfile               # Docker image that includes Tomcat + WAR
├── Jenkinsfile              # Jenkins pipeline for build & deploy
├── pom.xml                  # Maven multi-module project
├── server/                  # Java backend module (optional)
├── webapp/                  # Web application module
│   └── target/yourapp.war   # WAR file generated after Maven build
```

---

## 🚀 CI/CD Pipeline (via Jenkins)

### ✔️ Pipeline Steps

1. **Checkout Code** from GitHub
2. **Build WAR** using Maven
3. **Rename WAR** to `ROOT.war` (default Tomcat app)
4. **Build Docker Image** with Tomcat and WAR
5. **Push Image** to DockerHub *(optional)*
6. **Run Docker Container** mapping to port `8081`

---

## 🐳 Dockerfile

```dockerfile
FROM tomcat:11.0
RUN rm -rf /usr/local/tomcat/webapps/*
COPY ROOT.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
```

---

## 🛠 Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'yourdockerhubusername/tomcat-webapp'
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/your-username/your-repo.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Prepare WAR') {
            steps {
                sh 'cp webapp/target/*.war ROOT.war'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                        echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                        docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker stop tomcat-app || true
                    docker rm tomcat-app || true
                    docker run -d --name tomcat-app -p 8081:8080 $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        success {
            echo "App deployed at http://<your-server-ip>:8081"
        }
        failure {
            echo "Build or deployment failed"
        }
    }
}
```

---

## 🌐 Accessing the Application

1. Run:
    ```bash
    curl ifconfig.me
    ```
2. Then access:
    ```
    http://<your-server-ip>:8081
    ```

---

## 🔐 Notes

- Ensure port `8081` is open in your firewall or security group
- WAR should be named `ROOT.war` and copied properly in the Dockerfile

---

## 📦 Requirements

- Docker
- Jenkins
- Maven
- Java 11/17
- Git

---

## ✨ Credits

- [Apache Tomcat](https://tomcat.apache.org/)
- [Docker](https://www.docker.com/)
- [Jenkins](https://www.jenkins.io/)
- [Maven](https://maven.apache.org/)

---

## 📬 Need Help?

If you encounter any issues, feel free to raise an issue or get in touch.
