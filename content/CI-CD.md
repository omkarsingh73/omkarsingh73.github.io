## 1. What is CI/CD?

### Continuous Integration (CI)

Every code commit triggers:

```text
Code Commit
    ↓
Build
    ↓
Unit Test
    ↓
Code Quality Scan
    ↓
Artifact Creation
```

### Continuous Delivery/Deployment (CD)

```text
Artifact
   ↓
Deploy DEV
   ↓
Deploy QA
   ↓
Deploy PROD
```

**Goal:** Faster, reliable, repeatable releases.

---

# 2. Spring Boot CI/CD Architecture

```text
Developer
   ↓
GitHub/GitLab
   ↓
Jenkins/GitHub Actions
   ↓
Maven Build
   ↓
JUnit Tests
   ↓
SonarQube Scan
   ↓
JAR Creation
   ↓
Docker Build
   ↓
Docker Registry
   ↓
Kubernetes
   ↓
Production
```

---

# 3. Spring Boot Project Structure

```text
order-service

├── src/main/java
├── src/main/resources
│   ├── application.yml
│   ├── application-dev.yml
│   ├── application-qa.yml
│   └── application-prod.yml
│
├── Dockerfile
├── Jenkinsfile
├── pom.xml
└── kubernetes
    ├── deployment.yaml
    └── service.yaml
```

---

# 4. pom.xml (Build Configuration)

### Purpose

- Dependency management
    
- Packaging application
    
- Running tests
    
- Static analysis
    

Example:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>
```

---

# 5. Maven Commands Used in CI

### Compile

```bash
mvn compile
```

### Run Unit Tests

```bash
mvn test
```

### Package JAR

```bash
mvn clean package
```

Output:

```text
target/order-service.jar
```

### Skip Tests

```bash
mvn clean package -DskipTests
```

(Not recommended in CI)

---

# 6. Environment Configuration

## application.yml

```yaml
spring:
  application:
    name: order-service

server:
  port: 8080
```

Common configuration.

---

## application-dev.yml

```yaml
spring:
  datasource:
    url: jdbc:mysql://dev-db:3306/orders
```

---

## application-qa.yml

```yaml
spring:
  datasource:
    url: jdbc:mysql://qa-db:3306/orders
```

---

## application-prod.yml

```yaml
spring:
  datasource:
    url: jdbc:mysql://prod-db:3306/orders
```

---

# 7. Spring Profiles

Activate profile:

```bash
java -jar app.jar --spring.profiles.active=prod
```

Interview point:

> Build once, deploy everywhere. Only configuration changes per environment.

---

# 8. Unit Testing

Example:

```java
@SpringBootTest
class OrderServiceTest {

   @Test
   void createOrder() {
      assertTrue(true);
   }
}
```

Pipeline runs:

```bash
mvn test
```

Purpose:

- Catch defects early
    
- Prevent broken deployments
    

---

# 9. SonarQube Integration

Command:

```bash
mvn sonar:sonar
```

Checks:

- Bugs
    
- Vulnerabilities
    
- Code smells
    
- Coverage
    

Typical Quality Gate:

```text
Coverage > 80%
Critical Issues = 0
```

---

# 10. Docker Configuration

## Dockerfile

```dockerfile
FROM eclipse-temurin:17-jre

COPY target/order-service.jar app.jar

ENTRYPOINT ["java","-jar","app.jar"]
```

Build Image:

```bash
docker build -t order-service:v1 .
```

Run:

```bash
docker run -p 8080:8080 order-service:v1
```

---

# 11. Registry Push

Example:

```bash
docker tag order-service:v1 registry/order-service:v1

docker push registry/order-service:v1
```

Purpose:

Store deployable images.

Examples:

- Docker Hub
    
- Harbor
    
- Nexus
    
- Artifactory
    

---

# 12. Jenkins Pipeline

```groovy
pipeline {

 agent any

 stages {

   stage('Build') {
      steps {
         sh 'mvn clean package'
      }
   }

   stage('Test') {
      steps {
         sh 'mvn test'
      }
   }

   stage('Sonar') {
      steps {
         sh 'mvn sonar:sonar'
      }
   }

   stage('Docker Build') {
      steps {
         sh 'docker build -t app:${BUILD_NUMBER} .'
      }
   }

   stage('Push') {
      steps {
         sh 'docker push app:${BUILD_NUMBER}'
      }
   }

   stage('Deploy') {
      steps {
         sh 'kubectl apply -f deployment.yaml'
      }
   }
 }
}
```

---

# 13. Kubernetes Deployment

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: order-service

spec:
  replicas: 3

  template:
    spec:
      containers:
      - name: order-service
        image: order-service:v1
```

Purpose:

- High availability
    
- Scaling
    
- Rolling updates
    

---

# 14. Spring Boot Actuator

Dependency:

```xml
spring-boot-starter-actuator
```

Configuration:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health
```

Health URL:

```bash
/actuator/health
```

Response:

```json
{
  "status":"UP"
}
```

---

# 15. Readiness Probe

```yaml
readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
```

Purpose:

```text
Can application receive traffic?
```

If failed:

```text
Traffic blocked
```

---

# 16. Liveness Probe

```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
```

Purpose:

```text
Is application alive?
```

If failed:

```text
Pod restarted
```

---

# 17. Secrets Management

Bad:

```yaml
password: admin123
```

Good:

```yaml
password: ${DB_PASSWORD}
```

Kubernetes Secret:

```yaml
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password
```

---

# 18. Database Migration

Use Flyway.

Migration file:

```sql
V1__create_order_table.sql
```

Benefits:

- Version control for DB changes
    
- Automated schema updates
    
- Rollback support
    

---

# 19. Deployment Strategies

### Rolling Update

```text
Old Pods → New Pods
One by one
```

Advantages:

- No downtime
    
- Default Kubernetes strategy
    

---

### Blue-Green

```text
Blue = Current

Green = New

Switch Traffic
```

Advantages:

- Instant rollback
    

---

### Canary

```text
10% Traffic → New Version

90% Traffic → Old Version
```

Advantages:

- Lower deployment risk
    

---

# 20. Production Checklist

Before deployment:

✅ Unit Tests Pass

✅ Integration Tests Pass

✅ SonarQube Quality Gate Pass

✅ Security Scan Pass

✅ Docker Image Built

✅ Image Stored in Registry

✅ Kubernetes Manifest Validated

✅ Health Checks Configured

✅ Secrets Externalized

✅ Monitoring Enabled

---

# Senior Interview Answer (2 Minutes)

> "In our Spring Boot CI/CD process, developers commit code to Git. Jenkins triggers a Maven build, executes JUnit tests, and runs SonarQube quality analysis. The application is packaged as a JAR and containerized using Docker. Images are scanned for vulnerabilities and pushed to a registry. Kubernetes deploys the image using rolling updates. Spring Boot Actuator provides readiness and liveness endpoints for health validation. Environment-specific settings are managed through Spring Profiles, secrets are stored externally, and database schema changes are managed using Flyway. This enables automated, secure, and zero-downtime deployments."

## One-Page Interview Cheat Sheet

```text
Git
 ↓
Jenkins
 ↓
mvn clean package
 ↓
mvn test
 ↓
SonarQube
 ↓
JAR
 ↓
Docker Build
 ↓
Docker Registry
 ↓
Kubernetes
 ↓
Rolling Update
 ↓
Actuator Health Checks
 ↓
Production

Key Topics:
- Spring Profiles
- Maven Build
- JUnit Testing
- SonarQube
- Docker
- Kubernetes
- Actuator
- Readiness Probe
- Liveness Probe
- Flyway
- Secrets Management
- Rolling Update
- Blue-Green
- Canary Deployment
```

# 3. Common CI/CD Tools

|Area|Tools|
|---|---|
|SCM|Git, GitHub, GitLab|
|CI|Jenkins, GitLab CI, GitHub Actions|
|Artifact|Nexus, Artifactory|
|Build|Maven, Gradle, npm|
|Container|Docker|
|Orchestration|Kubernetes|
|GitOps|Argo CD, Flux|
|Security|SonarQube, Trivy, Snyk|
|Monitoring|Prometheus, Grafana|

## Q4. Explain Blue-Green Deployment.

```text
Production → Blue

Deploy New Version → Green

Test Green

Switch Traffic

Blue → Standby
```

Benefits:

- Zero downtime
    
- Easy rollback
    

---

## Q5. Explain Canary Deployment.

```text
10% users → New Version
90% users → Old Version

Monitor

50%

100%
```

Benefits:

- Reduced risk
    
- Real-user validation
    

Tools:

- Istio
    
- Argo Rollouts
    
- NGINX Ingress
    

---
## Q7. What are quality gates?

Examples:

### SonarQube

```text
Coverage > 80%

Critical Bugs = 0

Security Vulnerabilities = 0
```

Pipeline stops if gate fails.

---

## Q8. How do you optimize slow pipelines?

### Parallel Execution

```yaml
test:
 parallel: 4
```

### Cache Dependencies

```yaml
cache:
 paths:
   - .m2/
```

### Incremental Builds

### Build Agents Autoscaling

---

## Q9. Explain GitOps.

Git is source of truth.

```text
Git
 ↓
 ArgoCD
 ↓
 Kubernetes
```

Benefits:

- Auditable
    
- Reproducible
    
- Easy rollback
    

---

## Q10. Difference between Continuous Delivery and Continuous Deployment?

### Continuous Delivery

```text
Build → Test → Approval → Production
```

Manual approval before production.

### Continuous Deployment

```text
Build → Test → Production
```

No human intervention.

---

# 5. Scenario-Based Questions

## Scenario 1

Production deployment failed.

What do you do?

Answer:

1. Stop rollout
    
2. Analyze logs
    
3. Rollback
    
4. Root cause analysis
    
5. Fix and redeploy
    

---

## Scenario 2

Pipeline suddenly takes 60 mins instead of 15 mins.

Check:

- Build logs
    
- Agent utilization
    
- Network latency
    
- Dependency downloads
    
- Test execution time
    

---

## Scenario 3

SonarQube passes but production crashes.

Possible reasons:

- Environment mismatch
    
- Runtime configuration issue
    
- Missing secrets
    
- Infrastructure issue
    

---

## Scenario 4

How would you design CI/CD for 100 microservices?

Answer:

- Shared pipeline templates
    
- Reusable Jenkins libraries
    
- Central artifact repository
    
- GitOps deployment
    
- Policy enforcement
    
- Automated security scanning
    

---

# 6. Jenkins Interview Questions

## Declarative vs Scripted Pipeline

### Declarative

```groovy
pipeline {
 agent any

 stages {
   stage('Build') {
      steps {
        sh 'mvn clean package'
      }
   }
 }
}
```

### Scripted

```groovy
node {
   stage('Build') {
      sh 'mvn clean package'
   }
}
```

Declarative is preferred.

---

## Master-Agent Architecture

```text
Jenkins Controller
        ↓
     Agents
```

Benefits:

- Scalability
    
- Isolation
    
- Parallel execution
    

---

## Shared Libraries

```groovy
@Library('shared-lib')
```

Used for:

- Reusable code
    
- Standardization
    
- Governance
    

---

# 7. Kubernetes CI/CD Questions

## Rolling Update

```bash
kubectl rollout status deployment app
```

### Benefits

- Zero downtime
    
- Controlled rollout
    

---

## Readiness vs Liveness

### Readiness

Can pod receive traffic?

### Liveness

Is pod healthy?

Example:

```yaml
readinessProbe:
  httpGet:
    path: /health
```

---

# 8. Security Questions

## SAST vs DAST

### SAST

Before deployment.

Analyzes source code.

Examples:

- SonarQube
    
- Checkmarx
    

### DAST

After deployment.

Tests running application.

Examples:

- OWASP ZAP
    

---

## Container Security

Use:

```bash
trivy image myapp
```

Check:

- CVEs
    
- Misconfigurations
    
- Secrets
    

---

## 1. Deep-Dive: Enterprise CI/CD Configuration Best Practices

At an architectural level, modern CI/CD configuration goes beyond writing a single `.github/workflows/main.yml` file. It relies on three core tenets:

- **Immutable Infrastructure & Configuration Sync:** Code artifacts (e.g., Docker images, Helm charts) are built once and promoted through environments without modification.
    
- **Decoupled Pipelines:** Splitting application code pipelines from Infrastructure-as-Code (IaC) pipelines to reduce the blast radius of a failure.
    
- **GitOps Reconciliation:** Transitioning from "push-based" pipelines (e.g., standard Jenkins/GitLab executing commands against a cluster) to "pull-based" synchronization (e.g., ArgoCD or Flux tracking Git states inside the cluster), preventing configuration drift.
    

## 2. Architectural Interview Questions (10+ Years Experience)

### Q1: How do you secure a distributed CI/CD pipeline against Software Supply Chain attacks (e.g., SolarWinds style)?

- **The Intent:** Tests your awareness of modern DevSecOps, zero-trust infrastructure, and artifact validation.
    
- **Architectural Answer:**
    
    - **Pipeline Hardening:** Enforce short-lived, identity-based authentication (using **OIDC / OpenID Connect**) between pipeline runners and cloud providers instead of long-lived static IAM keys or passwords.
        
    - **Artifact Integrity:** Implement **Sigstore/Cosign** to cryptographically sign container images immediately after building them. Configure production clusters (via Kyverno or OPA Gatekeeper) to reject unsigned images.
        
    - **Dependency Governance:** Inject automated Software Bill of Materials (**SBOM**) generation (like Syft) and vulnerabilities scanners (**Trivy / Snyk**) directly into the containerization phase to halt builds that introduce critical CVEs.
        

### Q2: Imagine a shared enterprise monorepo with 50+ microservices. How do you design an efficient CI/CD pipeline that doesn't bottleneck engineering?

- **The Intent:** Tests your ability to scale systems and optimize resource consumption across giant teams.
    
- **Architectural Answer:**
    
    - **Change-Path Filtering:** Use native YAML path filters (e.g., `on.push.paths` in GitHub Actions) so a commit inside `/services/auth-api` only triggers its specific pipeline, bypassing the other 49 services.
        
    - **Dynamic Pipeline Generation:** For highly complex repositories, use tools like GitLab's Dynamic Child Pipelines or build engines like **Bazel / Nx** to analyze the dependency graph, rebuilding and re-testing only the affected code fragments.
        
    - **Self-Hosted Autoscaling Runners:** Deploy runner pools inside a Kubernetes cluster utilizing **Actions Runner Controller (ARC)** or Karpenter, scaling the worker nodes up during high morning PR volumes and shrinking them to zero at night.
        

### Q3: How do you orchestrate Zero-Downtime database migrations alongside application rollouts in a microservices ecosystem?

- **The Intent:** Evaluates your coordination of stateful database changes with stateless application changes.
    
- **Architectural Answer:** You must enforce a strict **Expand/Contract (Parallel Changes) pattern** across separate deployment cycles. A breaking schema change cannot happen in one step.
    
    1. **Expand Phase:** Run a pipeline that applies a schema change to _add_ the new column or table without removing the old one. The database must temporarily support both structural variations.
        
    2. **Code Rollout:** Deploy the new application version that writes to the new structure but can fall back to the old structure if rolled back.
        
    3. **Contract Phase:** Once the new application code is proven stable in production, a final, decoupled pipeline is manually or systematically run to clean up and delete the old database column/table.
        

### Q4: We are migrating from a push-based CD pattern to a pull-based GitOps approach. What are the engineering tradeoffs?

- **The Intent:** Tests high-level strategic decision-making and practical operational awareness.
    
- **Architectural Answer:**
    
    - **The Upside:** Git becomes the absolute single source of truth. Security improves dramatically because your external CI system no longer needs cluster-admin credentials injected into it; the GitOps controller (like ArgoCD) runs natively inside the private network cluster and pulls changes. It naturally resolves manual cluster changes ("configuration drift") by rewriting them back to the state defined in Git.
        
    - **The Downside:** Debugging becomes asynchronous and inherently more difficult. Developers lose visibility because log traces are split between the CI tool output and the Kubernetes cluster events. Managing secrets requires specialized tools like Bitnami Sealed Secrets or external Secret Store CSI drivers, adding layer complexity.
        

## 3. The Enterprise CI/CD Tooling Cheat Sheet (2026 Landscape)

This cheat sheet acts as an architectural blueprint for selecting, configuring, and optimizing modern delivery systems.

### Tool Selection Matrix

|Architecture / Paradigm|Core Stack Selection|Best For|Architectural Notes|
|---|---|---|---|
|**GitOps / Pull-Based**|Argo CD + Flux|Kubernetes-native multi-cluster scale|Pulls declarative state from Git; eliminates cluster credential exposure in CI.|
|**SaaS Pipeline-as-Code**|GitHub Actions / GitLab CI|High-velocity feature teams|YAML native, great dynamic matrices, excellent ecosystem integrations.|
|**Advanced Extensible**|Tekton / Jenkins (Groovy)|Legacy migration / Bespoke compliance|High management overhead but total control over containerized steps.|

### The DevSecOps Security Shield

Use these security integrations within your pipeline configuration stages:

- **Secrets Auditing:** `GitLeaks` or `TruffleHog` run during the pre-commit or initialization stage to block pipelines if raw credentials are leak-pushed.
    
- **Static Application Security Testing (SAST):** `Semgrep` or `SonarQube` to analyze code flaws prior to compiling.
    
- **Container & Infrastructure Scanning:** `Trivy` or `Checkov` to scan both the resulting Docker images and the Terraform IaC manifests for vulnerabilities before applying.
    

### Production Deployment Strategy Playbook

- **Blue/Green Deployment:** Two identical infrastructure stacks (Blue = Active, Green = Idle). The pipeline deploys to Green, runs integration tests, and swings the Load Balancer/DNS route. Safe, but doubles infrastructure costs.
    
- **Canary Deployment:** Code is pushed to a fractional subset of live instances (e.g., 2% of traffic). Automated metric analyzers (Prometheus/Datadog) track HTTP 5xx errors and latency. If anomalies are found, an automated rollback triggers instantly.
    
    Medium
    
- **Rolling Updates:** Gradually replaces old instances with new ones step-by-step. Standard in Kubernetes (`maxSurge` / `maxUnavailable`), low cost, but requires backwards-compatibility of components.
    

### Pipeline Configuration Blueprint (Anatomy of a Multi-Stage Config)

Below is a conceptual layout for structuring advanced enterprise pipelines. Focus on caching patterns and strict job dependencies to maximize efficiency:

YAML

```groovy
# Conceptual High-Performance Pipeline Architecture
stages:
  - static_analysis
  - single_build
  - dynamic_security
  - continuous_delivery

run_lint_and_unit_tests:
  stage: static_analysis
  strategy: parallel_matrix  # Run tests across multiple versions simultaneously
  cache_paths:
    - ~/.npm_or_pip_cache    # Retain dependency caches across iterations

compile_and_package:
  stage: single_build
  requires: [static_analysis]
  output_artifact: docker_image_v1.0.0
  action:
    - cosign sign --key prov.key my-registry/app:v1.0.0 # Sign your artifact

vulnerability_triage:
  stage: dynamic_security
  requires: [single_build]
  action:
    - trivy image my-registry/app:v1.0.0 # Scan the actual compiled artifact

promote_to_staging:
  stage: continuous_delivery
  requires: [dynamic_security]
  deployment_strategy: gitops_pr_trigger # Commit new version tag to config repo
```

Jenkin File


``` groovy

pipeline {

    agent any

    environment {
        REGISTRY = "docker.io/company"
        IMAGE_NAME = "myapp"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/company/myapp.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Scan') {
            steps {
                sh '''
                sonar-scanner \
                -Dsonar.projectKey=myapp \
                -Dsonar.sources=src
                '''
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                docker build \
                -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Image Scan') {
            steps {
                sh """
                trivy image \
                ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Push Image') {
            steps {
                sh """
                docker push \
                ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy') {
            steps {
                sh """
                kubectl set image deployment/myapp \
                myapp=${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Verify') {
            steps {
                sh '''
                kubectl rollout status deployment/myapp
                '''
            }
        }
    }
}

```

| Stage        | Intent                     |
| ------------ | -------------------------- |
| Checkout     | Get latest code            |
| Build        | Compile application        |
| Unit Test    | Validate functionality     |
| SonarQube    | Code quality               |
| Quality Gate | Governance                 |
| Docker Build | Create immutable artifact  |
| Trivy        | Security validation        |
| Push Image   | Artifact storage           |
| Deploy       | Release application        |
| Verify       | Confirm successful rollout |
| Rollback     | Fast recovery              |
