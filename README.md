# SonarQube

SonarQube is an open-source tool that ensures continuous code quality and security for twenty-seven (27) programming languages.

This tool provides a detailed report of bugs, code smells, vulnerabilities, code duplications, and it supports these languages by built-in rulesets and can also be extended with various plugins.

## Install SonarQube with docker-compose 

Take the following steps to install SonarQube with docker-compose:

### Setting up Firewall rules

- Open up port 9000 on your machine

```bash
sudo ufw allow 9000/tcp
```

- Verify the state of your firewall

```bash
sudo ufw status verbose
```

### Installing Docker and Docker-compose

- Install docker

```bash
sudo apt-get install docker -y
```

- Install docker-compose

```bash
sudo apt-get install docker-compose -y
```

### Creating a docker-compose file

- We create a docker-compose.yml file to contain docker commands to install SonarQube and PostgreSQL.

```bash
version: "3"

services:
  sonarqube:
    image: sonarqube:6.7.1
    container_name: sonarqube
    restart: unless-stopped
    environment:
      - POSTGRES_USER=<unique-user-name>
      - POSTGRES_PASSWORD=<unique-password>
      - SONARQUBE_JDBC_URL=jdbc:postgresql://db:5432/sonarqube
    ports:
      - "9000:9000"
      - "9092:9092"
    volumes:
      - sonarqube_conf:/opt/sonarqube/conf
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_bundled-plugins:/opt/sonarqube/lib/bundled-plugins

  db:
    image: postgres:10.1
    container_name: db
    restart: unless-stopped
    environment:
      - POSTGRES_USER=<unique-user-name>
      - POSTGRES_PASSWORD=<unique-password>
      - POSTGRES_DB=sonarqube
    volumes:
      - sonarqube_db:/var/lib/postgresql10
      - postgresql_data:/var/lib/postgresql10/data

volumes:
  postgresql_data:
  sonarqube_bundled-plugins:
  sonarqube_conf:
  sonarqube_data:
  sonarqube_db:
  sonarqube_extensions:
```

- Run the following command to start up your docker-compose file

```bash
sudo docker-compose up -d 
```

**Caveat:** Peradventure you run into a *Could not connect to Docker-daemon* error, it implies you have to add *Docker group* to the current user by running the following command ```sudo usermod -aG docker $USER```

- Ensure SonarQube is running by checking your logs

```bash
sudo docker-compose logs
```

**Access SonarQube UI via:**  <http://your_SonarQube_publicdns_name:9000/>

## Integrate a Jenkins pipeline to the build

- Connect to your AWS EC2 instance, then run the following commands, to download and install Jenkins.

```bash
sudo yum update –y
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum upgrade
sudo yum install jenkins java-1.8.0-openjdk-devel -y
sudo systemctl daemon-reload
sudo systemctl status jenkins
```

### Configure Jenkins

- To view Jenkins UI, type `http://<your_server_public_DNS>:8080` in your browser.

![image](https://user-images.githubusercontent.com/49791498/133862137-b0c6f9c2-46ab-45dc-a7dd-eea8b158f465.png)

- Take the steps highlighted [in this article](https://www.blazemeter.com/blog/how-to-integrate-your-github-repository-to-your-jenkins-project) to integrate your Github repository into your Jenkins pipeline.

- Type the following command to display the Administrator password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

- On the **Customize Jenkins page**, click **install suggested plugins.**

- Once the installation is complete, create a **First Admin User**, click **Save** and **Continue**.

- On the **Manage Jenkins** page, click on **Manage Plugins**.

- Under the **Available** tab, search for **SonarQube Scanner** and install it.

### Add Tokens

- On your SonarQube Admin account, click on the **My profile** option.

- Under **Security**, generate a token under the **Generate Tokens** option.

- On Jenkins, head to the **Manage Jenkins** page and click on **Configure System**.

- Scroll down to the **SonarQube Scanner** section and fill the prompts as shown below, click **apply** and then **save**.
<pictures>

- Further configurations depending on the programming language of choice, can be found in [this article](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-jenkins/).


## using a terraform script

