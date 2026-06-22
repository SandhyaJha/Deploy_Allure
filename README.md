# Allure Docker Service Deployment Guide (AlmaLinux 8)

## Objective

Deploy a centralized Allure reporting server for automated test results and make reports available through a web interface.

Current report URL:

http://<server_ip>:5050/allure-docker-service/projects/<my_project>/reports/latest/index.html

---

## Environment

* OS: AlmaLinux 8
* Server IP: <server_ip>
* Reporting Tool: Allure Docker Service
* Container Runtime: Docker
* Project Name: <my_project>

---

## Step 1: Install Docker

Configure Docker repository:

```bash
sudo dnf config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
```

Install Docker:

```bash
sudo dnf install -y \
docker-ce \
docker-ce-cli \
containerd.io \
docker-buildx-plugin \
docker-compose-plugin
```

Start Docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Verify installation:

```bash
docker --version
docker compose version
```

---

## Step 2: Create Allure Server Directory Structure

```bash
sudo mkdir -p /opt/allure-server

cd /opt/allure-server
```

Create Docker Compose configuration.

---

## Step 3: Create docker-compose.yml

```yaml
services:
  allure:
    image: frankescobar/allure-docker-service:latest
    container_name: allure-server

    ports:
      - "5050:5050"

    environment:
      CHECK_RESULTS_EVERY_SECONDS: 5
      KEEP_HISTORY: 1
      KEEP_HISTORY_LATEST: 30

    restart: unless-stopped
```

---

## Step 4: Start Allure Server

```bash
cd /opt/allure-server

docker compose up -d
```

Verify container status:

```bash
docker ps
```

Expected container:

```text
allure-server
```

---

## Step 5: Configure Firewall

Allow access to Allure web interface:

```bash
sudo firewall-cmd --permanent --add-port=5050/tcp
sudo firewall-cmd --reload
```

Verify:

```bash
sudo firewall-cmd --list-ports
```

---

## Step 6: Verify Allure Service

Open:

```text
http://<server_ip>:5050
```

Verify service health:

```bash
curl http://<server_ip>:5050/projects
```

---

## Step 7: Create Project

Create a project called <my_project>:

```bash
curl -X POST \
"http://<server_ip>:5050/projects" \
-H "Content-Type: application/json" \
-d '{"id":"<my_project>"}'
```

Verify:

```bash
curl http://<server_ip>:5050/projects
```

---

## Step 8: Generate Allure Results

Install Allure Pytest plugin:

```bash
pip install allure-pytest
```

Execute tests:

```bash
pytest tests \
--alluredir=allure-results
```

Generated files:

```text
allure-results/
├── *.json
├── *.txt
└── attachments/
```

---

## Step 9: Upload Results to Allure Server

Upload all generated result files:

```bash
for f in allure-results/*; do
    curl -X POST \
    "http://<server_ip>:5050/send-results?project_id=<my_project>" \
    -F "files[]=@$f"
done
```

Generate report:

```bash
curl -X GET \
"http://<server_ip>:5050/generate-report?project_id=<my_project>"
```

---

## Step 10: View Reports

Latest report:

```text
http://<server_ip>:5050/allure-docker-service/projects/<my_project>/reports/latest/index.html
```

Specific report versions:

```text
http://<server_ip>:5050/allure-docker-service/projects/<my_project>/reports/1/index.html

http://<server_ip>:5050/allure-docker-service/projects/<my_project>/reports/2/index.html
```

---

## Jenkins Integration Example

```groovy
stage('Run Tests') {
    steps {
        sh '''
        pytest tests --alluredir=allure-results
        '''
    }
}

stage('Publish Allure Results') {
    steps {
        sh '''
        for f in allure-results/*; do
            curl -X POST \
            "http://<server_ip>:5050/send-results?project_id=<my_project>" \
            -F "files[]=@$f"
        done

        curl -X GET \
        "http://<server_ip>:5050/generate-report?project_id=<my_project>"
        '''
    }
}
```

---

## Benefits

* Centralized reporting server
* Historical report retention
* Multiple project support
* No Jenkins plugins required
* Accessible from any browser
* Suitable for automated regression and validation pipelines
