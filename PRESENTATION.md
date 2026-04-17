# Spring Boot Email Service - CI/CD Journey
## Project Presentation

---

## 📋 Agenda

1. Project Overview
2. Spring Boot Application Architecture
3. Application Features & Code
4. Docker Containerization
5. CI/CD Pipeline Implementation
6. GitHub Actions Workflow
7. Key Achievements
8. Next Steps

---

## 🎯 Project Overview

**Project Name:** Spring Boot Email Service

**Objective:** Build a complete cloud-native application with automated CI/CD deployment pipeline

**Technology Stack:**
- Backend: Spring Boot 3.2.0
- Build Tool: Maven
- Container: Docker
- CI/CD: GitHub Actions
- Registry: Docker Hub
- Language: Java 17

---

## 🏗️ Application Architecture

### Three-Tier Architecture

```
┌─────────────────────────────────────┐
│   REST API Layer                    │
│   EmailController                   │
│   POST /email/send                  │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│   Business Logic Layer              │
│   EmailService                      │
│   - Email composition               │
│   - Service integration             │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│   Third-Party Integration           │
│   JavaMailSender (SMTP)             │
│   Gmail / Mail Provider             │
└─────────────────────────────────────┘
```

---

## 💻 Code Structure

### Project Layout
```
springboot-email-service/
├── pom.xml
├── Dockerfile
├── sonar-project.properties
├── .github/
│   └── workflows/
│       └── ci.yml
└── src/
    └── main/
        ├── java/com/example/email/
        │   ├── EmailServiceApplication.java
        │   ├── EmailController.java
        │   └── EmailService.java
        └── resources/
            └── application.properties
```

---

## 📧 EmailServiceApplication.java

**Entry Point of the Application**

```java
@SpringBootApplication
public class EmailServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(EmailServiceApplication.class, args);
    }
}
```

- Bootstraps Spring Boot application
- Enables auto-configuration
- Starts embedded Tomcat server on port 8080

---

## 🎮 EmailController.java

**REST API Endpoint**

```java
@RestController
@RequestMapping("/email")
public class EmailController {
    
    @Autowired
    private EmailService emailService;
    
    @PostMapping("/send")
    public String sendEmail(
        @RequestParam String to,
        @RequestParam String subject,
        @RequestParam String body) {
        emailService.sendEmail(to, subject, body);
        return "Email sent successfully!";
    }
}
```

**Endpoint:**
- `POST /email/send?to=example@gmail.com&subject=Hello&body=Message`

---

## 📤 EmailService.java

**Business Logic Implementation**

```java
@Service
public class EmailService {
    
    @Autowired
    private JavaMailSender mailSender;
    
    public void sendEmail(String to, String subject, String body) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(to);
        message.setSubject(subject);
        message.setText(body);
        message.setFrom("your-email@gmail.com");
        mailSender.send(message);
    }
}
```

**Responsibilities:**
- Composes email message
- Handles SMTP communication
- Manages mail delivery

---

## ⚙️ Configuration (application.properties)

```properties
server.port=8080

# SMTP Configuration
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=your-email@gmail.com
spring.mail.password=your-app-password
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
```

---

## 🐳 Docker Strategy

### Multi-Stage Build Approach

**Stage 1: Builder** (Compile)
```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests
```

**Stage 2: Runtime** (Lightweight)
```dockerfile
FROM eclipse-temurin:17-jre-jammy
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Benefits:**
- ✅ Final image: ~200MB (only JRE, no Maven)
- ✅ Build image: ~500MB (can be discarded after build)
- ✅ Faster pulls and deployments

---

## 🔄 CI/CD Pipeline Architecture

### Five-Stage Pipeline

```
┌──────────────┐
│   Build & Test   │ ← Compile, Unit tests,
└────────┬─────┘   ← Coverage reports
         │
         ▼
┌──────────────────────┐
│ Code Quality Analysis│ ← SonarQube scan,
└────────┬─────────────┘   ← Security checks
         │
         ▼
┌──────────────────────┐
│  Build Docker Image  │ ← Compile JAR,
└────────┬─────────────┘   ← Create image,
         │               ← Push to Docker Hub
         ▼
┌──────────────────────┐
│  Deploy to Prod      │ ← Pull image,
└────────┬─────────────┘   ← Verify deployment
         │
         ▼
┌──────────────────────┐
│ Notify on Failure    │ ← Alert if any step fails
└──────────────────────┘
```

---

## 📝 GitHub Actions Workflow - Build & Test

**Job: `build`**

```yaml
build:
  name: Build and Test
  runs-on: ubuntu-latest
  
  steps:
    - name: Checkout code
    - name: Set up JDK 17
    - name: Build with Maven
      run: mvn clean package -DskipTests
    - name: Run Tests
      run: mvn test
    - name: Generate Coverage Report
      run: mvn jacoco:report
    - name: Upload test results
    - name: Upload coverage reports
```

**Outputs:**
- test-results (JUnit reports)
- coverage-reports (JaCoCo coverage)

---

## 🔍 GitHub Actions Workflow - Code Quality

**Job: `code-quality`** (depends on: build)

```yaml
code-quality:
  name: Code Quality Analysis
  runs-on: ubuntu-latest
  needs: build
  
  steps:
    - name: Checkout code
    - name: Set up JDK 17
    - name: SonarQube Scan
      uses: SonarSource/sonarcloud-github-action@master
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

**Security Checks:**
- Code vulnerabilities
- Code smells
- Test coverage
- Dependency scanning

---

## 🏗️ GitHub Actions Workflow - Docker Build

**Job: `build-docker`** (depends on: build)

```yaml
build-docker:
  name: Build Docker Image
  runs-on: ubuntu-latest
  needs: build
  if: github.ref == 'refs/heads/main'
  
  steps:
    - name: Checkout code
    - name: Set up Docker Buildx
    - name: Log in to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    - name: Extract metadata
    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        push: true
        tags: |
          ${{ secrets.DOCKER_USERNAME }}/springboot-email-service:latest
          ${{ secrets.DOCKER_USERNAME }}/springboot-email-service:${{ github.sha }}
```

**Registry:** docker.io (Docker Hub)

---

## 🚀 GitHub Actions Workflow - Deploy

**Job: `deploy`** (depends on: build-docker)

```yaml
deploy:
  name: Deploy to Production
  runs-on: ubuntu-latest
  needs: build-docker
  if: github.ref == 'refs/heads/main'
  
  steps:
    - name: Log in to Docker Hub
    - name: Pull Docker image
      run: docker pull ${{ secrets.DOCKER_USERNAME }}/springboot-email-service:latest
    - name: Display deployment info
      run: |
        echo "Image: docker.io/${{ secrets.DOCKER_USERNAME }}/springboot-email-service:latest"
        echo "Run: docker run -p 8080:8080 ..."
```

**Extensible to:**
- AWS ECS
- Azure Container Instances
- Kubernetes
- DigitalOcean App Platform

---

## 📊 GitHub Actions Workflow - Notifications

**Job: `notify-failure`** (runs on failure)

```yaml
notify-failure:
  name: Notify on Failure
  runs-on: ubuntu-latest
  needs: [build, build-docker]
  if: failure()
  
  steps:
    - name: Send failure notification
      run: |
        echo "❌ Pipeline failed for commit: ${{ github.sha }}"
        echo "Repository: ${{ github.repository }}"
        echo "Branch: ${{ github.ref }}"
        echo "Check: ${{ github.server_url }}/..."
```

**Can be extended to:**
- Slack notifications
- Email alerts
- PagerDuty incident creation
- Webhook callbacks

---

## 🔑 Required GitHub Secrets

All secrets configured in GitHub **Settings > Secrets and variables > Actions**:

| Secret | Purpose | Example |
|--------|---------|---------|
| `DOCKER_USERNAME` | Docker Hub login | `your-docker-username` |
| `DOCKER_PASSWORD` | Docker Hub token | `dckr_pat_xxxx` |
| `SONAR_TOKEN` | SonarQube analysis | `squ_xxxxx` |
| `GITHUB_TOKEN` | Auto-provided by GitHub | Automatic |

---

## ✨ Key Achievements

### 1️⃣ **Application Development**
- ✅ RESTful API for email delivery
- ✅ Spring Boot best practices
- ✅ Dependency injection & services
- ✅ Configuration management

### 2️⃣ **Containerization**
- ✅ Multi-stage Docker builds
- ✅ Optimized image size (~200MB)
- ✅ Docker Hub integration

### 3️⃣ **CI/CD Pipeline**
- ✅ Automated build & test
- ✅ Code quality gates (SonarQube)
- ✅ Docker image building & pushing
- ✅ Production deployment ready
- ✅ Failure notifications

### 4️⃣ **DevOps Practices**
- ✅ Infrastructure as Code (IaC)
- ✅ Version control integration
- ✅ Automated testing
- ✅ Security scanning
- ✅ Artifact management

---

## 📈 Pipeline Execution Flow

```
Developer pushes code to main branch
           │
           ▼
GitHub detects push
           │
           ▼
Trigger: build-test job
   ├─ Checkout
   ├─ Setup JDK 17
   ├─ Maven build
   ├─ Run tests
   └─ Generate coverage
           │
           ▼
Trigger: code-quality job (parallel)
   ├─ Setup JDK 17
   └─ SonarQube scan
           │
           ▼
Trigger: build-docker job
   ├─ Setup Docker Buildx
   ├─ Login to Docker Hub
   ├─ Build Docker image
   └─ Push image with tags
           │
           ▼
Trigger: deploy job
   ├─ Pull image
   └─ Display deployment info
           │
           ▼
✅ All jobs complete OR ❌ Notify on failure
```

---

## 🆕 Running the Application

### Locally (with Maven)
```bash
mvn spring-boot:run
```

### With Docker
```bash
docker run -p 8080:8080 \
  -e SPRING_MAIL_USERNAME=your-email@gmail.com \
  -e SPRING_MAIL_PASSWORD=your-app-password \
  yourusername/springboot-email-service:latest
```

### Testing the Endpoint
```bash
curl -X POST "http://localhost:8080/email/send?to=recipient@gmail.com&subject=Test&body=Hello"
```

---

## 🎓 DevOps Concepts Implemented

| Concept | Implementation |
|---------|-----------------|
| **Containerization** | Docker multi-stage builds |
| **CI/CD** | GitHub Actions workflow |
| **Code Quality** | SonarQube integration |
| **Testing** | JUnit + JaCoCo coverage |
| **Security** | Docker secrets, SonarQube scan |
| **Artifact Management** | Docker Hub registry |
| **Infrastructure as Code** | YAML workflow definition |
| **Monitoring** | Build/deployment notifications |

---

## 🚀 Next Steps & Enhancements

### Phase 2 - Advanced Deployment
- [ ] Kubernetes deployment (YAML manifests)
- [ ] Helm charts for package management
- [ ] AWS ECS/Fargate integration
- [ ] Load balancing & auto-scaling

### Phase 3 - Monitoring & Logging
- [ ] ELK Stack (Elasticsearch, Logstash, Kibana)
- [ ] Prometheus metrics
- [ ] Grafana dashboards
- [ ] Distributed tracing (Jaeger)

### Phase 4 - Advanced Features
- [ ] Email templates
- [ ] Scheduled email delivery
- [ ] Async processing (RabbitMQ/Kafka)
- [ ] Email retry logic
- [ ] Rate limiting

---

## 📚 Learning Outcomes

By completing this project, you've learned:

✅ **Spring Boot Fundamentals**
- REST API development
- Dependency injection
- Auto-configuration

✅ **Docker Containerization**
- Dockerfile best practices
- Multi-stage builds
- Image optimization

✅ **CI/CD with GitHub Actions**
- Workflow automation
- Job dependencies
- Secrets management

✅ **DevOps Practices**
- Infrastructure as Code
- Automated testing
- Security scanning
- Deployment automation

✅ **Software Engineering**
- Version control workflows
- Code quality standards
- Documentation practices

---

## 🎉 Project Summary

### Timeline
- **Phase 1:** Spring Boot development ✅
- **Phase 2:** Docker containerization ✅
- **Phase 3:** CI/CD pipeline setup ✅
- **Phase 4:** Code quality integration ✅
- **Phase 5:** Production deployment ready ✅

### Metrics
- **Code Quality:** Scanned by SonarQube
- **Test Coverage:** Generated by JaCoCo
- **Deployment:** Automated via GitHub Actions
- **Reliability:** Multi-stage enforcement

### Repository
- **URL:** https://github.com/MrunalSPatil/EmailNotificationService
- **Main Branch:** Protected with CI/CD checks
- **Docker Hub:** yourusername/springboot-email-service

---

## 🙏 Thank You!

**Questions?**

For detailed implementation:
- 📄 Review `.github/workflows/ci.yml`
- 📄 Check `Dockerfile` for containerization
- 📄 See `pom.xml` for dependencies
- 📄 Examine `sonar-project.properties` for quality config

**Contact:**
- GitHub: MrunalSPatil
- Repository: EmailNotificationService

---
