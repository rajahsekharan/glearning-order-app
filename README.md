# Secure CI Pipeline for Java Application with OIDC & AWS ECR Integration

This project implements a secure CI pipeline using GitHub Actions, OpenID Connect (OIDC), and AWS ECR to build and push a Dockerized Java application without using static AWS credentials.

---

## Pipeline Design

The CI pipeline is defined in `.github/workflows/build-and-push.yml` and consists of the following steps:

1. **Trigger**: Runs on push to the `main` branch.
2. **Checkout Code**: Retrieves the Java source code.
3. **OIDC Authentication**: Uses GitHub OIDC to assume an AWS IAM Role securely (no secrets).
4. **Docker Build**: Uses a multi-stage Dockerfile to build a lightweight image.
5. **Push to ECR**: Pushes the built image to a private AWS Elastic Container Registry (ECR).

---

## Multi-Stage Docker Build

The Dockerfile uses two stages:

- **Stage 1: Build Stage**
  - Base Image: `maven:3.8.5-openjdk-17`
  - Builds the JAR using Maven: `mvn clean package -DskipTests`
  
- **Stage 2: Runtime Stage**
  - Base Image: `openjdk:17-jdk-slim`
  - Copies only the built JAR into the runtime container for optimal size and security.

This keeps the final image lean and production-ready.

---

## IAM Role Setup for OIDC

To avoid using long-lived AWS credentials, GitHub Actions assumes an IAM Role via OpenID Connect:

- **Role Name**: `GitHubActionsECRPushRole`
- **Account ID**: `838482266459`
- **Trust Policy**:
  - Identity Provider: `token.actions.githubusercontent.com`
  - Condition: `sub` must match `repo:<github-username>/<repo-name>:ref:refs/heads/main`
- **Permissions Policy**:
  - Minimal ECR permissions (`ecr:GetAuthorizationToken`, `ecr:PutImage`, etc.)

This ensures the workflow has just enough access to push Docker images.

---

## GitHub Actions Workflow

File: `.github/workflows/build-and-push.yml`

Key actions used:

- `actions/checkout@v3`
- `aws-actions/configure-aws-credentials@v2` (OIDC role assumption)
- `aws ecr get-login-password` to authenticate Docker
- `docker build`, `tag`, and `push` to ECR

---

## 📸 Screenshots

- GitHub Actions run: successful OIDC authentication, Docker build, and image push.
- AWS ECR: Image visible in the private ECR repository with `latest` tag.

---

## Challenges Faced

- **OIDC Setup**: Understanding the trust relationship conditions and IAM permissions.
- **Minimal Permissions**: Ensuring least privilege while still allowing Docker image pushes.
- **Docker Optimization**: Using multi-stage builds for better performance and smaller images.

---

## Output Image

Pushed Docker image to:

838482266459.dkr.ecr.ap-south-1.amazonaws.com/glearning-order-app:latest


---

## Technologies Used

- Java 17 (Spring Boot)
- Maven
- Docker (multi-stage)
- GitHub Actions
- AWS ECR
- IAM OIDC

---

## Conclusion

This implementation follows best practices for CI/CD security by avoiding static secrets and using OIDC-based role assumption. The Docker image is lean, production-ready, and securely delivered to a private container registry.

