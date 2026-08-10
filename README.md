# DevSecOps CI/CD Pipeline

A practical **DevSecOps project** that takes a Python Flask application from source code to a production server using **GitHub Actions, Docker, Docker Hub, and automated security scanning**.

The main goal of this project is to demonstrate how security can be integrated directly into a CI/CD pipeline instead of being treated as a separate step.


##  Application

The project is a simple **Python Flask web application** with two endpoints:

* `/` — serves the web application
* `/health` — returns a basic health-check response

The application runs using **Gunicorn** inside a lightweight Python Alpine Docker image.

The application itself is intentionally simple so the focus remains on the **DevOps and DevSecOps workflow**.



#  Architecture

```text
                         Developer
                             │
                             │ git push / workflow dispatch
                             ▼
                    ┌──────────────────┐
                    │    GitHub Repo   │
                    └────────┬─────────┘
                             │
                             ▼
                ┌──────────────────────────┐
                │  DevSecOps Orchestrator  │
                │ devsecops-pipline.yml    │
                └────────────┬─────────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
   Code Quality        Secrets Scan      Dependency Scan
   Flake8/Bandit          GitLeaks           pip-audit
          │                  │                  │
          └──────────────────┼──────────────────┘
                             │
                             ▼
                      Dockerfile Scan
                         Hadolint
                             │
                             ▼
                    ┌─────────────────┐
                    │ Build Container │
                    │  & Push to      │
                    │   Docker Hub    │
                    └────────┬────────┘
                             │
                             ▼
                    Trivy Image Scan
                    HIGH / CRITICAL CVEs
                             │
                             ▼
                    ┌─────────────────┐
                    │ Production      │
                    │ Server / EC2    │
                    └────────┬────────┘
                             │
                             ▼
                       Docker Compose
                             │
                             ▼
                       Flask Application
```

The workflow is orchestrated by `.github/workflows/devsecops-pipline.yml`, which calls the individual reusable workflows in sequence.



##  Security Checks

| Stage               | Tool            | What it checks                            |
| ------------------- | --------------- | ----------------------------------------- |
| Code quality / SAST | Flake8 + Bandit | Python code                               |
| Secret scanning     | GitLeaks        | Credentials and secrets                   |
| Dependency scanning | pip-audit       | Vulnerable Python packages                |
| Dockerfile scanning | Hadolint        | Dockerfile best practices                 |
| Image scanning      | Trivy           | HIGH / CRITICAL container vulnerabilities |

- ### Code & SAST

  Bandit is used to perform static security analysis on the Python application.

- ### Secrets

  GitLeaks scans the repository for accidentally committed credentials, API keys, tokens, and other sensitive information.

- ### Dependencies

  `pip-audit` checks the Python packages defined in `requirements.txt` for known vulnerabilities.

- ### Dockerfile

  Hadolint validates the Dockerfile before the image is built.

- ### Container Image

  After the image is built, **Trivy** scans it for vulnerabilities. The pipeline is configured to fail for **HIGH** and **CRITICAL** findings.


##  Build & Deployment

Once the initial checks pass, GitHub Actions builds the Docker image and pushes it to Docker Hub.

The image is tagged using:

```text
latest
branch name
Git commit SHA
```

The commit SHA provides a direct connection between a Git commit and the Docker image that was deployed.

After the Trivy scan succeeds, the deployment workflow:

1. Connects to the production server through SSH.
2. Copies the Docker Compose configuration.
3. Logs into Docker Hub.
4. Pulls the required image.
5. Restarts the application using Docker Compose.



##  Project Structure

```text
DevSecOps/
│
├── .github/workflows/
│   ├── devsecops-pipline.yml    # Pipeline orchestrator
│   ├── code-quality.yml         # Linting + SAST
│   ├── secrets-scan.yml         # GitLeaks
│   ├── dependency-scan.yml      # pip-audit
│   ├── docker-lint.yml          # Hadolint
│   ├── build-and-push.yml       # Docker build + Docker Hub
│   ├── image-scan.yml           # Trivy
│   └── deploy-to-server.yml     # Production deployment
│
├── app.py
├── index.html
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
└── README.md
```

---

##  GitHub Secrets

The pipeline uses GitHub Secrets for sensitive credentials:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN

EC2_SSH_HOST
EC2_SSH_USERNAME
EC2_SSH_PRIVATE_KEY
```

These credentials are used for Docker Hub authentication and SSH-based deployment. They are **not stored directly in the repository**.



## Run Locally

### Using Python

```bash
git clone https://github.com/Duryab333/DevSecOps.git
cd DevSecOps

python3 -m venv venv
source venv/bin/activate

pip install -r requirements.txt
python app.py
```

### Using Docker

```bash
docker build -t devsecops-app .
docker run -p 80:80 devsecops-app
```

The application will be available at:

```text
http://localhost
```




#  Technologies Used

### Application

* Python
* Flask
* Gunicorn

### Containerization

* Docker
* Docker Compose
* Docker Hub
* Python Alpine image

### CI/CD

* GitHub Actions
* Reusable workflows

### DevSecOps / Security

* Bandit
* GitLeaks
* pip-audit
* Hadolint
* Trivy

### Infrastructure / Deployment

* Linux server / EC2
* SSH
* Docker Compose



##  What This Project Demonstrates

This project brings together several real-world DevOps/DevSecOps practices:

* CI/CD with GitHub Actions
* Reusable workflows
* Shift-left security
* SAST and secret scanning
* Dependency vulnerability scanning
* Docker security
* Container image scanning
* Secure credential management
* Docker image versioning
* Automated production deployment

The main idea is simple:

**Don't just build and deploy the application — validate and secure it throughout the delivery process.**

