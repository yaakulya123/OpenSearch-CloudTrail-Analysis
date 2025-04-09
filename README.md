# My OpenSearch Deployment for CloudTrail Analysis POC

![image](https://github.com/user-attachments/assets/a4a6b147-ac9f-4c1e-933c-1251b67145b0)

This is a documentation of how I deployed OpenSearch on a virtual machine for analyzing AWS CloudTrail logs. I wanted to create an easy-to-follow guide for anyone trying to replicate this setup.



## What I Needed

- A virtual machine (I used Kali Linux on UTM on my MacBook Pro)
- Docker and Docker Compose

## The Working Installation Process

I followed these steps to successfully deploy OpenSearch using Docker:

### Step 1: Install and Configure Docker

```bash
# Install Docker and Docker Compose
sudo apt install docker.io docker-compose -y

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add my user to the docker group to avoid permission issues
sudo usermod -aG docker $USER

# Apply new group membership without logging out
newgrp docker
```

### Step 2: Create Project Directory and Configuration

```bash
# Create a directory for the OpenSearch project
mkdir -p ~/opensearch-poc/data
cd ~/opensearch-poc

# Create docker-compose.yml file
nano docker-compose.yml
```

I added this content to the docker-compose.yml file:

```yaml
version: '3'
services:
  opensearch:
    image: opensearchproject/opensearch:latest
    container_name: opensearch
    environment:
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m"
      - "DISABLE_SECURITY_PLUGIN=true"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    volumes:
      - ./data:/usr/share/opensearch/data
    ports:
      - 9200:9200
      - 9600:9600

  opensearch-dashboards:
    image: opensearchproject/opensearch-dashboards:latest
    container_name: opensearch-dashboards
    ports:
      - 5601:5601
    environment:
      - 'OPENSEARCH_HOSTS=["http://opensearch:9200"]'
      - "DISABLE_SECURITY_DASHBOARDS_PLUGIN=true"
    depends_on:
      - opensearch
```

### Step 3: Launch OpenSearch with Docker Compose

```bash
# Start the containers
docker-compose up -d
```

This command pulled the necessary Docker images and started both the OpenSearch engine and OpenSearch Dashboards.


![image](https://github.com/user-attachments/assets/bf33b92d-6857-43f2-9969-06d02cf8f3fe)

### Step 4: Verify the Installation

I confirmed the installation was successful by:

```bash
# Check running containers
docker ps
```

And by accessing:
- OpenSearch API: http://localhost:9200
- OpenSearch Dashboards: http://localhost:5601

![image](https://github.com/user-attachments/assets/6f1d39af-7b13-4a09-b9b3-159c016f9bec)

---

![image](https://github.com/user-attachments/assets/6846c062-5893-4d2c-aaa6-8460d0f69362)



## Challenges I Faced and How I Solved Them? 

### Challenge 1: Failed Package Installation Approach

Before settling on Docker, I initially tried installing OpenSearch using the Debian package manager:

```bash
# Update system packages
sudo apt update

# Install Java (required for OpenSearch)
sudo apt install default-jdk -y

# Install dependencies
sudo apt install curl -y
sudo apt install gnupg2 -y

# Add OpenSearch repository key
wget -qO - https://artifacts.opensearch.org/publickeys/opensearch.pgp | sudo apt-key add -

# Add OpenSearch repository
echo "deb https://artifacts.opensearch.org/releases/bundle/opensearch/2.x/apt stable main" | sudo tee /etc/apt/sources.list.d/opensearch-2.x.list

# Update package lists
sudo apt update

# Install OpenSearch
sudo apt install opensearch -y
```

This approach failed with an error:
```
ERROR: Something went wrong during demo configuration installation. Please see the logs in /var/log/opensearch/install_demo_configuration.log
```

I decided to switch to Docker instead, which provided a cleaner and more reliable installation.

### Challenge 2: Docker Permission Issues

When I first tried to run Docker Compose, I got this error:
```
PermissionError: [Errno 13] Permission denied
```

I solved this by adding my user to the Docker group and applying the changes:
```bash
sudo usermod -aG docker $USER
newgrp docker
```

This allowed me to run Docker commands without sudo permissions.

## Next Steps

For the full POC with CloudTrail log analysis, the next steps would be preparing sample data and building dashboards, but for now I've able to successfully complete the deployment of OpenSearch in a controlled VM environment. If any errors or warning caused, please feel free to pulll up a request and I can look up at that!
