# SonarQube

SonarQube is an open-source tool that ensures continuous code quality and security for twenty-seven (27) programming languages.

This tool provides a detailed report of bugs, code smells, vulnerabilities, code duplications, and it supports these languages by built-in rulesets and can also be extended with various plugins.

## Install SonarQube with docker-compose on an EC2 instance

In setting up your EC2 instance, open up the following additional ports:

```bash
port 8080/TCP
port 9000/TCP
```

### Installing Docker and Docker-compose

- Install docker

```bash
sudo yum update -y
sudo yum install docker
sudo service docker start
sudo usermod -a -G docker ec2-user
```

- Exit the container via the ```exit``` command and log back in, to enable the docker service run without the **sudo** command.

- Install docker-compose

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
```

### Creating a docker-compose file

- Create a docker-compose.yml file to contain docker commands to install SonarQube and PostgreSQL.

```bash
nano docker-compose.yml
```

to contain:

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

The command `Ctrl + X` and `Y`, closes the script.

- Run the following command to start up your docker-compose file

```bash
docker-compose up -d 
```

![ter1](https://user-images.githubusercontent.com/49791498/134591430-b6ca93aa-5c66-404a-b31f-98445ce64e3c.png)

- Ensure SonarQube is running by checking your logs

```bash
sudo docker-compose logs
```

**Access SonarQube UI via:**  <http://your_SonarQube_publicdns_name:9000/>
**Default login details are 'admin', 'admin' for both username and pasword.**
**[Installing Docker](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html), [and docker-compose](https://acloudxpert.com/how-to-install-docker-compose-on-amazon-linux-ami/)**

## Integrate a Jenkins pipeline to the build

- While SonarQube is running, then run the following commands in your VM to download and install Jenkins.

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

- Open up Jenkins in your browser, by typing in: `http://<your_server_public_DNS>:8080` in the search bar.

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

## Creating a Terraform script