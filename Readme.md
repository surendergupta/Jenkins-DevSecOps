# jenkins DevSecOps

## Create EC2 Instance
- InstanceType: t2.large
- Storage: 25GB

## Installation

- First System Update

```
sudo apt-get update
```

- Install Docker

```
sudo apt-get install docker.io -y
sudo apt-get install docker-compose -y
sudo usermod -aG docker $USER
sudo usermod -aG docker jenkins
sudo chmod 666 /var/run/docker.sock

```

- Install Java

```
sudo apt install fontconfig openjdk-17-jre -y
java -version

```

- Install Jenkins

```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo usermod -aG docker jenkins

sudo reboot

systemctl status jenkins

```

- Add 8080 Port in Security Group
![alt text](./images/image.png)

- Steps to configure Jenkins

![alt text](./images/image-1.png)

![alt text](./images/image-2.png)

![alt text](./images/image-3.png)

- Select Suggested Plugins

![alt text](./images/image-4.png)

![alt text](./images/image-5.png)

![alt text](./images/image-6.png)

![alt text](./images/image-7.png)

![alt text](./images/image-8.png)

- Jenkins Setup Completed

- Dashboard

![alt text](./images/image-8.png)

- Manage Jenkins

![alt text](./images/image-10.png)

- Plugins

![alt text](./images/image-11.png)

- Available Plugins

![alt text](./images/image-9.png)

- Install Pugins

- Restart Jenkins

- Install SonarQube

```
docker run -itd --name sonarqube-server -p 9000:9000 sonarqube:lts-community
```

![alt text](./images/image-12.png)

- Add 9000 Port in Security Group

![alt text](./images/image-13.png)

![alt text](./images/image-14.png)

- Update Old Password

![alt text](./images/image-15.png)

- Menu -> Adminitration -> Configuration -> Webhook -> create Webhook

- Craete Webhook for jenkins to sonarqube exchange information

![alt text](./images/image-16.png)

![alt text](./images/image-17.png)

![alt text](./images/image-18.png)

![alt text](./images/image-19.png)

![alt text](./images/image-20.png)

![alt text](./images/image-21.png)

- Copy Token

```
squ_167b4fbcef326ff2f96a59efa28df50ca07531b2
``` 

![alt text](./images/image-22.png)

![alt text](./images/image-23.png)

![alt text](./images/image-24.png)

![alt text](./images/image-25.png)

- Install Trivy

```
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
trivy --version

```

- Create Pipeline

![alt text](./images/image-26.png)

- Jenkins pipeline Script

```
pipeline{
    agent any
    environment{
        SONAR_HOME = tool "Sonar-scaner"
    }
    
    stages{
        stage("Clone Code from GitHub"){
            steps{
                git url: "https://github.com/krishnaacharyaa/wanderlust.git", branch: "devops"
            }
        }
        stage("SonarQube Quality Analysis"){
            steps{
                withSonarQubeEnv("Sonar"){
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=wanderlust -Dsonar.projectKey=wanderlust"
                }
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Sonar Quality Gate Scan"){
            steps{
                timeout(time: 2, unit: "MINUTES"){
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage("Trivy File System Scan"){
            steps{
                sh "trivy fs --format  table -o trivy-fs-report.html ."
            }
        }
        stage("Deploy using Docker compose"){
            steps{
                sh "docker-compose up -d"
            }
        }
    }
}
```


![alt text](image.png)

- Add Port 5173 and 5000 in Security Group