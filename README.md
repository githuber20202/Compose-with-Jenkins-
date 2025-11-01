# Jenkins CI Infrastructure - Containerized Setup

![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat&logo=docker&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-LTS-D24939?style=flat&logo=jenkins&logoColor=white)
![CI](https://img.shields.io/badge/CI-Continuous%20Integration-success?style=flat)
![Infrastructure](https://img.shields.io/badge/Infrastructure-as%20Code-orange?style=flat)

## 📋 Overview

A **containerized Jenkins infrastructure** setup for Continuous Integration (CI) workflows. This project provides a Jenkins environment with master-agent architecture for learning and development purposes, designed to run automated build and test pipelines.

**This repository focuses on the CI infrastructure layer:**
- **Containerized Jenkins Master** - Central orchestration and UI
- **Jenkins Agent(s)** - Distributed build executors
- **Docker-in-Docker capability** - Build Docker images within pipelines
- **Persistent storage** - Jenkins configuration and build history
- **Network isolation** - Secure container communication

**Key Features:**
- Jenkins master-agent architecture for scalable builds
- Docker Compose orchestration for easy deployment
- Custom Jenkins image with pre-installed tools (Python, Git, Docker CLI)
- Persistent volumes for data retention
- Ready for CI pipeline execution

---

## 📦 Prerequisites

Before starting, ensure you have:

- **Docker Engine** 20.10+ ([Install Docker](https://docs.docker.com/get-docker/))
- **Docker Compose** 2.0+ (included with Docker Desktop)
- **Git** for version control
- **4GB RAM minimum** (8GB recommended)
- **Operating System**: Linux, macOS, or Windows with WSL2

**Verify installations:**
```bash
docker --version
docker compose version
git --version
```

---

## 🚀 Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/githuber20202/Compose-with-Jenkins-.git
cd Compose-with-Jenkins-
```

### 2. Start Jenkins Environment
```bash
docker compose up -d
```

### 3. Access Jenkins UI
Open your browser and navigate to:  
👉 **http://127.0.0.1:8081**

### 4. Get Initial Admin Password
```bash
docker exec jenkins-master cat /var/jenkins_home/secrets/initialAdminPassword
```

### 5. Complete Setup Wizard
- Paste the admin password
- Install **recommended plugins**
- Create your **admin user**
- Start building your first pipeline! 🎯

---

## 🏗️ Architecture

```
┌─────────────────────────────────┐
│     Jenkins Master              │
│     (Port 8081)                 │
│  - Web UI                       │
│  - Pipeline Orchestration       │
│  - Docker CLI Access            │
└────────────┬────────────────────┘
             │
        ┌────┴─────┐
        │ Network  │
        │ Bridge   │
        └────┬─────┘
             │
┌────────────┴────────────────────┐
│     Jenkins Agent               │
│  - Build Executor               │
│  - Docker CLI Access            │
│  - Workspace: jenkins_agent/    │
└─────────────────────────────────┘
             │
        ┌────┴─────┐
        │  Docker  │
        │  Socket  │
        └──────────┘
```

---

## 📁 Project Structure

```
jenkins-lab/
├── docker-compose.yml      # Defines Jenkins master & agent services
├── Dockerfile.jenkins      # Custom Jenkins image with Docker CLI
├── jenkinsfile            # Example CI/CD pipeline
├── .gitignore             # Excludes jenkins_home/ and jenkins_agent/
├── jenkins_home/          # Jenkins persistent data (auto-created)
├── jenkins_agent/         # Agent workspace (auto-created)
└── README.md              # This documentation
```

**Key Files:**
- `docker-compose.yml` → Orchestrates Jenkins master and agent containers
- `Dockerfile.jenkins` → Builds custom Jenkins image with Python, Git, and Docker CLI
- `jenkinsfile` → Complete CI/CD pipeline example (build → push → GitOps update)

---

## 🐳 Docker CLI Installation Explained

The `Dockerfile.jenkins` includes a critical section that installs Docker CLI inside the Jenkins container:

```dockerfile
# Install Docker CLI
RUN curl -fsSL https://get.docker.com -o get-docker.sh && \
    sh get-docker.sh && \
    rm get-docker.sh
```

### What Does This Do?

This command performs three operations in sequence:

1. **`curl -fsSL https://get.docker.com -o get-docker.sh`**
   - Downloads Docker's official installation script from `https://get.docker.com`
   - Saves it as `get-docker.sh` in the container
   - Flags explained:
     - `-f` (fail silently) - Don't show error page on HTTP errors
     - `-s` (silent) - Don't show progress bar
     - `-S` (show errors) - Show errors even in silent mode
     - `-L` (follow redirects) - Follow HTTP redirects if any

2. **`sh get-docker.sh`**
   - Executes the downloaded installation script
   - Installs Docker CLI tools (docker command) inside the Jenkins container
   - Detects the OS and installs the appropriate Docker version

3. **`rm get-docker.sh`**
   - Removes the installation script after use
   - Keeps the Docker image size smaller (cleanup best practice)

### Why Do We Need Docker CLI in Jenkins?

**The Problem:**
- Jenkins needs to build Docker images as part of CI pipelines
- Jenkins runs inside a Docker container itself
- By default, containers don't have Docker installed

**The Solution:**
- Install Docker CLI inside the Jenkins container
- Mount the host's Docker socket (`/var/run/docker.sock`) as a volume
- This allows Jenkins to communicate with the host's Docker daemon

**How It Works:**
```
┌─────────────────────────────────────┐
│   Jenkins Container                 │
│                                     │
│   ┌──────────────────┐             │
│   │  Docker CLI      │             │
│   │  (installed)     │             │
│   └────────┬─────────┘             │
│            │                        │
│            │ communicates via       │
│            │ /var/run/docker.sock   │
└────────────┼────────────────────────┘
             │
             ▼
┌─────────────────────────────────────┐
│   Host Machine                      │
│                                     │
│   ┌──────────────────┐             │
│   │  Docker Daemon   │             │
│   │  (actual engine) │             │
│   └──────────────────┘             │
└─────────────────────────────────────┘
```

**What This Enables:**
- ✅ Build Docker images from Jenkinsfile
- ✅ Push images to Docker Hub/registries
- ✅ Run Docker commands in pipeline stages
- ✅ Test containerized applications

**Example Pipeline Usage:**
```groovy
stage('Build Docker Image') {
    steps {
        script {
            // This works because Docker CLI is installed
            sh 'docker build -t myapp:latest .'
            sh 'docker push myapp:latest'
        }
    }
}
```

### Security Considerations

⚠️ **Important Notes:**

1. **Docker Socket Mounting**
   - Mounting `/var/run/docker.sock` gives Jenkins **full Docker access**
   - Jenkins can start/stop ANY container on the host
   - This is a **security risk** in production environments

2. **Alternative Approaches for Production:**
   - **Docker-in-Docker (DinD)** - Run Docker daemon inside container
   - **Kaniko** - Build images without Docker daemon
   - **Buildah** - Daemonless container builds
   - **Podman** - Rootless container engine

3. **Why We Use This Approach:**
   - ✅ Simple setup for learning
   - ✅ Fast builds (uses host's Docker cache)
   - ✅ No nested Docker daemon overhead
   - ⚠️ **NOT recommended for production**

### Verification

After the container starts, you can verify Docker CLI installation:

```bash
# Check Docker CLI is installed
docker exec jenkins-master docker --version

# Test Docker access
docker exec jenkins-master docker ps

# View Docker info
docker exec jenkins-master docker info
```

**Expected Output:**
```
Docker version 24.0.x, build xxxxx
```

If you see this output, Docker CLI is successfully installed and can communicate with the host's Docker daemon through the mounted socket.

---

## ⚠️ Security Considerations

**IMPORTANT: This setup is for LEARNING PURPOSES ONLY!**

### Security Warnings:
- ⚠️ **Root User**: Jenkins master runs as `root` for simplicity
- ⚠️ **Docker Socket**: `/var/run/docker.sock` is mounted (full Docker access)
- ⚠️ **No TLS**: HTTP only, no HTTPS encryption
- ⚠️ **Default Ports**: Using standard ports without firewall rules

### For Production Environments:
- ✅ Use proper user permissions (non-root)
- ✅ Avoid mounting Docker socket (use Docker-in-Docker or Kaniko)
- ✅ Enable HTTPS with valid certificates
- ✅ Implement network segmentation and firewall rules
- ✅ Use secrets management (HashiCorp Vault, AWS Secrets Manager)
- ✅ Enable audit logging and monitoring

---

## 💡 Usage Examples

### Create Your First Pipeline

1. **Navigate to Jenkins UI**: http://127.0.0.1:8081
2. Click **"New Item"**
3. Enter a name and select **"Pipeline"**
4. In the Pipeline section, choose **"Pipeline script from SCM"**
5. Configure your Git repository
6. Set **Script Path** to `jenkinsfile`
7. Click **"Save"** and **"Build Now"**

### Add More Agents

Edit `docker-compose.yml` and duplicate the agent service:

```yaml
jenkins-agent-2:
  image: jenkins/inbound-agent:latest
  container_name: jenkins-agent-2
  environment:
    - JENKINS_URL=http://jenkins-master:8080
    - JENKINS_AGENT_NAME=agent-2
    - JENKINS_AGENT_WORKDIR=/home/jenkins/agent
  volumes:
    - ./jenkins_agent_2:/home/jenkins/agent
    - /var/run/docker.sock:/var/run/docker.sock
  depends_on:
    - jenkins-master
  networks:
    - jenkins-network
  restart: unless-stopped
```

Then restart:
```bash
docker compose up -d
```

### View Logs

```bash
# Jenkins master logs
docker logs -f jenkins-master

# Agent logs
docker logs -f jenkins-agent-1

# All services
docker compose logs -f
```

### Stop Environment

```bash
# Stop containers (keeps data)
docker compose stop

# Stop and remove containers (keeps volumes)
docker compose down

# Remove everything including volumes
docker compose down -v
```

---

## 🔧 Troubleshooting

### Jenkins Won't Start

**Problem**: Container exits immediately or won't start

**Solutions**:
```bash
# Check if port 8081 is already in use
netstat -an | grep 8081  # Linux/Mac
netstat -ano | findstr 8081  # Windows

# Check Docker is running
docker ps

# View container logs
docker logs jenkins-master

# Restart Docker service
sudo systemctl restart docker  # Linux
# Or restart Docker Desktop on Mac/Windows
```

### Permission Denied on jenkins_home

**Problem**: `Permission denied` errors in logs

**Solution**:
```bash
# Fix ownership (Jenkins uses UID 1000)
sudo chown -R 1000:1000 jenkins_home/

# Or on Windows with WSL2
wsl sudo chown -R 1000:1000 jenkins_home/
```

### Agent Not Connecting

**Problem**: Agent shows as offline in Jenkins UI

**Solutions**:
1. Check agent logs: `docker logs jenkins-agent-1`
2. Verify network: `docker network inspect jenkins-lab_jenkins-network`
3. Restart agent: `docker compose restart jenkins-agent`
4. Check Jenkins master logs for connection errors

### Docker Commands Fail in Pipeline

**Problem**: `docker: command not found` in Jenkins pipeline

**Solution**:
- Verify Docker socket is mounted in `docker-compose.yml`
- Check Docker CLI is installed in the Jenkins image
- Run: `docker exec jenkins-master docker --version`

### Out of Disk Space

**Problem**: Jenkins fills up disk space

**Solution**:
```bash
# Clean old Docker images
docker system prune -a

# Clean Jenkins workspace
docker exec jenkins-master rm -rf /var/jenkins_home/workspace/*

# Check disk usage
docker system df
```

---

## 🔗 Related Repositories

This Jenkins infrastructure is part of a complete CI/CD workflow:

- **[JB-PROJECT](https://github.com/githuber20202/JB-PROJECT)** - Application source code (Python Flask)
- **[jb-gitops](https://github.com/githuber20202/jb-gitops)** - GitOps repository for CD (Helm charts + Argo CD)
- **[Compose-with-Jenkins](https://github.com/githuber20202/Compose-with-Jenkins-.git)** - This repository (CI infrastructure)

**Separation of Concerns:**
- **CI (This Repo)** → Jenkins infrastructure for building and testing
- **CD (GitOps Repo)** → Argo CD handles deployment to Kubernetes
- **Source Code** → Application being built and deployed

**What is GitOps?**

GitOps is a deployment methodology where:
- Infrastructure and application configurations are stored in Git
- Git serves as the single source of truth
- Automated tools (like Argo CD) sync Git state to the cluster
- Changes are made via Git commits, not manual kubectl commands

**In this architecture:**
1. **Jenkins (CI)** builds Docker images and pushes to registry
2. **Jenkins updates** the `values.yaml` file in the GitOps repository with the new image tag
3. **Argo CD (CD)** detects the Git commit and deploys automatically to Kubernetes
4. **Kubernetes** runs the updated application

**Why update values.yaml in Git?**

The `values.yaml` file in the GitOps repository contains Helm chart configuration, including:
```yaml
image:
  repository: formy5000/resources_viewer
  tag: "sha-abc123"  # ← Jenkins updates this line
```

When Jenkins builds a new Docker image, it:
1. Tags the image with the Git commit SHA (e.g., `sha-abc123`)
2. Pushes the image to Docker Hub
3. **Updates the `values.yaml` file** in the GitOps repo with the new tag
4. Commits and pushes the change to GitHub

This Git commit triggers Argo CD to:
- Detect the change in the GitOps repository
- Pull the new Helm values
- Deploy the updated application to Kubernetes

**Benefits of this approach:**
- ✅ **Separation of concerns** - CI builds, CD deploys
- ✅ **Git as source of truth** - All changes tracked in version control
- ✅ **Audit trail** - Every deployment has a Git commit
- ✅ **Easy rollbacks** - Revert the Git commit to rollback
- ✅ **No direct cluster access** - Jenkins doesn't need Kubernetes credentials

---

## 📚 What This Repository Provides

This repository focuses **exclusively on the CI infrastructure**:

**Included in This Repo:**
- ✅ Jenkins Master container configuration
- ✅ Jenkins Agent(s) setup for distributed builds
- ✅ Docker Compose orchestration
- ✅ Custom Jenkins image with build tools (Python, Git, Docker CLI)
- ✅ Network and volume configuration
- ✅ Example Jenkinsfile demonstrating:
  - Building Docker images
  - Pushing to Docker Hub
  - **Updating values.yaml in GitOps repository**

**NOT Included (Handled by Other Repos):**
- ❌ Application source code → See [JB-PROJECT](https://github.com/githuber20202/JB-PROJECT)
- ❌ Kubernetes deployment configs → See [jb-gitops](https://github.com/githuber20202/jb-gitops)
- ❌ Argo CD setup → Handled in GitOps repository
- ❌ Helm charts → Stored in GitOps repository

**CI vs CD Separation:**

| Component | Responsibility | Repository |
|-----------|---------------|------------|
| **Jenkins (CI)** | Build images, Push to registry, **Update values.yaml** | This Repo |
| **GitOps Repo** | Store Helm charts and values.yaml | jb-gitops |
| **Argo CD (CD)** | Sync Git → Kubernetes, Deploy | jb-gitops |
| **Application** | Source Code | JB-PROJECT |

**Jenkins Pipeline Flow:**
```
1. Checkout code from JB-PROJECT
2. Build Docker image
3. Tag image with Git SHA (e.g., sha-abc123)
4. Push image to Docker Hub
5. Clone GitOps repository (jb-gitops)
6. Update values.yaml with new image tag
7. Commit and push to GitOps repository
8. [Argo CD takes over from here]
```

**What Jenkins Does NOT Do:**
- ❌ Does not deploy to Kubernetes directly
- ❌ Does not need kubectl or cluster credentials
- ❌ Does not manage Helm releases
- ❌ Only updates the Git repository (GitOps principle)

**Technical Documentation:**
- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/)
- [Docker Compose Reference](https://docs.docker.com/compose/)
- [Jenkins Master-Agent Architecture](https://www.jenkins.io/doc/book/scaling/architecting-for-scale/)

---

## 🤝 Contributing

Contributions are welcome to improve this CI/CD pipeline implementation.

**How to contribute:**
- 🐛 **Report bugs** - Open an issue with detailed reproduction steps
- 💡 **Suggest improvements** - Propose enhancements to the pipeline
- 📝 **Improve documentation** - Submit PRs for README updates
- 🔧 **Add features** - Extend pipeline capabilities

**Guidelines:**
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -m 'Add some improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request with detailed description

---

## 📄 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

**TL;DR**: You can use, modify, and distribute this project freely, even for commercial purposes, as long as you include the original license.

---

## 👤 Author

**Alexander-Y** — DevOps Learning Journey 🚀

- GitHub: [@githuber20202](https://github.com/githuber20202)
- Project Link: [Compose-with-Jenkins](https://github.com/githuber20202/Compose-with-Jenkins-.git)

---

## 🙏 Acknowledgments

- Jenkins community for excellent documentation
- Docker team for containerization technology
- The DevOps community for sharing knowledge and best practices

---

**Production-Ready CI Infrastructure** 🚀

This repository provides the CI layer. For the complete CI/CD workflow, see the related repositories above.

If you found this implementation useful, please ⭐ star the repository!
