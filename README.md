# Jenkins Lab - CI/CD Learning

## Setup

1. Install **Docker** & **Docker Compose**
2. Clone this repo:
   ```bash
   git clone https://github.com/githuber20202/Compose-with-Jenkins-.git
   cd Compose-with-Jenkins-
   ```
3. Run Jenkins environment:
   ```bash
   docker compose up -d
   ```
4. Access Jenkins UI:  
   👉 http://127.0.0.1:8081
5. Get the initial admin password:
   ```bash
   docker exec jenkins-master cat /var/jenkins_home/secrets/initialAdminPassword
   ```
6. Follow the setup wizard in Jenkins:
   - Install recommended plugins  
   - Create your admin user  
   - Start building your first pipeline 🎯

---

## Structure

- `docker-compose.yml` → defines **Jenkins master** and **agent** containers  
- `Dockerfile.jenkins` → builds a custom Jenkins image with Docker socket access  
- `jenkins_home/` → Jenkins persistent data (gitignored)  
- `jenkins_agent/` → Jenkins agent workspace (gitignored)  
- `README.md` → this documentation  

---

## Notes

- Jenkins master can run Docker commands using the mounted `/var/run/docker.sock`.
- The agent automatically connects to the master container.
- You can add more agents by duplicating the service definition in `docker-compose.yml`.

---

## Next Steps

This stage covers **only the Jenkins setup** (local CI environment).  
Next stages will include:
- Building & pushing Docker images  
- Updating Helm chart values  
- Automating deployment via **Argo CD**  

---

## Author

Alex — DevOps Learning Journey 🚀
