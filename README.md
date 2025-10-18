# Jenkins Lab - CI/CD Learning

## Setup

1. Install Docker & Docker Compose
2. Clone this repo
3. Run:
```bash
   docker-compose up -d
```
4. Access Jenkins: http://127.0.0.1:8081
5. Get initial password:
```bash
   docker exec jenkins-master cat /var/jenkins_home/secrets/initialAdminPassword
```

## Structure

- `docker-compose.yml` - Jenkins master + agent setup
- `jenkins_home/` - Data directory (gitignored)
- `jenkins_agent/` - Agent workspace (gitignored)

## Author

Alex - DevOps Learning Journey :)
