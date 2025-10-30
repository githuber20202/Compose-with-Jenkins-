# Jenkins Lab - CI/CD Learning

![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat&logo=docker&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-LTS-D24939?style=flat&logo=jenkins&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=flat)
![Learning](https://img.shields.io/badge/Purpose-Learning-orange?style=flat)

## 📋 Overview

This is a **learning lab** for setting up a complete CI/CD pipeline using Jenkins, Docker, and GitOps practices. The project demonstrates how to build a containerized Jenkins environment with a **master-agent architecture**, enabling you to run distributed builds and learn modern DevOps workflows.

**What you'll learn:**
- Setting up Jenkins in Docker containers
- Configuring Jenkins master-agent architecture
- Building and pushing Docker images via Jenkins pipelines
- Implementing GitOps workflows with Argo CD
- Managing CI/CD infrastructure as code

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

This Jenkins lab is part of a complete CI/CD learning series:

- **[JB-PROJECT](https://github.com/githuber20202/JB-PROJECT)** - Python Flask application (source code)
- **[jb-gitops](https://github.com/githuber20202/jb-gitops)** - GitOps repository with Helm charts
- **[Compose-with-Jenkins](https://github.com/githuber20202/Compose-with-Jenkins-.git)** - This repository (Jenkins setup)

**Complete Workflow:**
1. **Code** → JB-PROJECT (application source)
2. **Build** → This Jenkins Lab (CI pipeline)
3. **Deploy** → jb-gitops (Argo CD + Kubernetes)

---

## 📚 Next Steps

This stage covers **only the Jenkins setup** (local CI environment).

**Upcoming stages:**
- ✅ **Stage 1**: Jenkins Setup (You are here!)
- 🔄 **Stage 2**: Building & pushing Docker images
- 🔄 **Stage 3**: Updating Helm chart values via GitOps
- 🔄 **Stage 4**: Automating deployment with **Argo CD**
- 🔄 **Stage 5**: Monitoring with Prometheus & Grafana

**Learning Resources:**
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [GitOps Principles](https://www.gitops.tech/)

---

## 🤝 Contributing

This is a **learning project** and contributions are welcome!

**How to contribute:**
- 🐛 **Report bugs** - Open an issue describing the problem
- 💡 **Suggest improvements** - Share your ideas for enhancements
- 📝 **Improve documentation** - Submit PRs for README updates
- 🎓 **Share your experience** - Help others learn from your journey

**Guidelines:**
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -m 'Add some improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

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

**Happy Learning! 🎓**

If you found this helpful, please ⭐ star the repository!
