# TOOLS INSTALLATION AND CONFIGURATION 

**Order of Setup Server Set-up**
- Jenkins-Controller/Master Server
- Nexus Server
- SonarQube Server
- Maven Server
- Gradle Server

**Some Usedata for Tools Configuration**
- jenkins-controller-agent-config: https://github.com/EngineerChris/jenkins-controller-agent-config/tree/main


## 1️.  Set up Jenkins-Controller Server (ec2)
- name: Jenkins-Controller
- machine type: t2.medium
- image/ami: Amazon Linux 2
- Key pair: Select or Create
- Security group ports: 8080, 22
 
 ### Pass As User Data or Login and Install Jenkins 
 ```bash
#!/bin/bash
sudo su
yum update –y
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
amazon-linux-extras install epel -y
amazon-linux-extras install java-openjdk11 -y
yum install jenkins -y
systemctl enable jenkins
systemctl start jenkins

## Installing Git
yum install git -y
 ```

## 2. Set-up Nexus EC2 Server: "NexusArtifactory"
- **name**: NexusArtifactory
- **machine type**: t2.medium
- **image/ami**: Amazon Linux 2
- **Key pair**: Select or Create
- **Security group ports**: 8081 (Allow traffic from anywhere 0.0.0.0/0), 22 (Allow traffic from anywhere 0.0.0.0/0)
- **user-data-in-the-advanced-details**: https://github.com/EngineerChris/DevOps-Tools-Installation-Configuration/blob/main/maven-sonarqube-nexus-jenkins-install/nexus-install.sh


## 3. Set-up SonarQube EC2 Server: "SonarQube-Analysis" 
- **name**: NexusArtifactory 
- **machine type**: t2.medium 
- **image/ami**: Ubuntu 20.4 
- **Key pair**: Select or Create 
- **Security group ports**: 9000 (Allow traffic from anywhere 0.0.0.0/0),  
 and SSH 22 (Allow traffic from anywhere 0.0.0.0/0) 
- **user-data-in-the-advanced-details**: https://github.com/EngineerChris/DevOps-Tools-Installation-Configuration/blob/main/maven-sonarqube-nexus-jenkins-install/sonarqube-install.sh 


## 4. Configure Nexus 
### Login to Nexus Server Web UI
  A) CREATE MAVEN PROJECT ARTIFACT REPOSITORY
  - Click on the Admin Repository Secition
    - Click on `Repositories`
    - Click on `Create Repository`
      - Select: `Maven2(hosted)` #SNAPSHOT as defined in POM.XML file line 13
      - Name: `maven-java-webapp-repository` # this name must match the repo name in the repo-url 
      - Click: `CREATE`

  B) CREATE GRADLE PROJECT ARTIFACT REPOSITORY 
  - Click on the Admin (gear) Repository Section 
    - Click on `Repositories`
    - Click on `Create Repository`
      - Name: `gradle-java-webapp-repository`
      - Select: `Maven2(hosted)`  #SNAPSHOT as defined in BUILD.GRADLE file line 12
      - Click: `CREATE`


## 5. UPDATE YOUR MAVEN CONFIGURATIONS 

### A) Update Maven (POM.xml & Settings.xml) File
  - Switch to the <'maven-sonarqube-nexus-jenkins'> project branch
  - Update The Nexus Private IP Address In These Files 
  - POM.xml: Line '64' and '68'
  - Settings.xml: Line '63' and '74'
  - Update The Nexus Repository As Well To Yours "if different"
  - Update the username and password on the settings.xml

### B) Update The Maven 'User Data Script' With Your 'settings.xml' GitHub RawLink 
 - Update the User data before Creating the Maven Build Instance/Env
 - Here: 
    - https://github.com/EngineerChris/DevOps-Tools-Installation-Configuration/blob/main/maven-sonarqube-nexus-jenkins-install/sonarqube-install.sh  OR  
    - https://github.com/awanmbandi/realworld-cicd-pipeline-project/blob/jenkins-declarative-master-client-confg/runbooks/maven-install.md#2%EF%B8%8F%E2%83%A3-install-and-configure-java11-and-apache-maven
 

### C) Add SonarQube Scanner Snippet Configurations In Maven Project 'Jenkinsfile' File 
 - Line: '31' - '38'

## 6. UPDATE YOUR GRADLE CONFIGURATIONS

### A) Update The Nexus PrivateIP Address & Repository Name For Gradle Project In The "build.gradle" File
 - Switch to the <'gradle-sonarqube-nexus-jenkins'>
 - Line: '57'

### B) Add The SonarQube Scanner Configurations API In The Gradle 'build.gradle' File
- Line: '37' - '44'

## 7. Set-up Maven Server: "Maven-Agent-JDK11"
- **name**: Maven-Agent-JDK11
- **machine type**: t2.mmicro
- **image/ami**: Amazon Linux 2
- **Key pair**: Select or Create
- **Security group ports**:  SSH port 22, Allow traffic from anywhere 0.0.0.0/0
- **user-data-in-the-advanced-details**: Use "maven-install-config.sh" in  the config folder of this repo and branch  https://github.com/EngineerChris/jenkins-controller-agent-config/blob/main/archive/maven-install.md  #update the settings.xml github path if not done already.
- ssh into Maven-Agent-Server: ssh -i kp.pem ec2-user@ip
  - check the maven version and details: mvn -v, which maven
  - **Test that Maven can be executed within this env**: mvn validate (Build failed error is ok at this point, no project is being built )

## 8. Set-up Gradle Server: "Gradle-Agent-JDK11"
- **name**: Gradle-Agent-JDK11
- **machine type**: t2.mmicro
- **image/ami**: Ubuntu 22.04
- **Key pair**: Select or Create
- **Security group ports**:  22, Allow traffic from anywhere 0.0.0.0/0
- **user-data-in-the-advanced-details**: https://github.com/EngineerChris/jenkins-controller-agent-config/blob/main/archive/gradle-install.md
- ssh into M=Gradle-Agent-Server: ssh -i kp.pem ec2-user@ip
  - check the gradle version and details: gradle, which gradle
- **Test that Gradle can be executed within this env**: mvn validate (Build failed error is ok at this point, no project is being built )

## 9. CONFIGURE MASTER & CLIENT CONFIGURATION
- Click on "Manage Jenkins" >> Click "Nodes and Cloud" >> Click "New Node"
- Click `New Node`
    - Node name: `Maven-Build-Env` 
    - Type: `Permanent Agent` >> Click `CREATE`

#### 9.1. Configure "Maven-Build-Env"
- Name:                  `Maven-Build-Env`
- Number of Executors:   `5` (for example, maximum jobs to execute at a time)
- Remote root directory: `/opt/maven-builds`# workspace that jenkins will use to store all  build executable generated by jenkins in the maven build env.
- Labels:                `Maven-Build-Env`
- Usage:                 `Use this node as much as possible`
- Launch method:         `Launch agents via SSH`
    - Host:   `Provide IP of Maven-Build-Server`
    - Credentials:
        - Click on `Add / Jenkins` and Select `Username and Password`
        - Username: `jenkinsmaster`
        - Password: `jenkinsmaster`
        - ID: `Maven-Build-Env-Credential`
        - Save
        - Credentials: Select `Maven-Build-Env-Credential`
    - Host Key Verification Strategy: `Non Verifying Verification Strategy`
    - Availability: `Keep this agent online as much as possible`
- NODE PROPERTIES
    - Select `Environment variables`
        - Click `Add`
        - 1st Variable:
            - Name: `MAVEN_HOME`
            - Value: `/usr/share/apache-maven`
        - 2nd Variable:
            - Name: `PATH`
            - Value: `$MAVEN_HOME/bin:$PATH`

    - Click `SAVE`
- NOTE: Make sure the `Agent Status` shows `Agent successfully connected and online` on the Logs
- NOTE: Repeat the process for adding additional Nodes

#### 9.2. Configure "Gradle-Build-Env"
- Click `New Node`
    - Node name: `Gradle-Build-Env` 
    - Type: `Permanent Agent` >> Click `CREATE`

- Name:                  `Gradle-Build-Env`
- Number of Executors:   `5` (for example, maximum jobs to execute at a time)
- Remote root directory: `/opt/gradle-builds`
- Labels:                `Gradle-Build-Env`
- Usage:                 `Use this node as much as possible`
- Launch method:         `Launch agents via SSH`
    - Host:   `Provide IP of Gradle-Build-Server`
    - Credentials:
        - Click on `Add / Jenkins` and Select `Username and Password`
        - Username: `jenkinsmaster`
        - Password: `jenkinsmaster`
        - ID: `Gradle-Build-Env-Credential`
        - Save
        - Credentials: Select `Gradle-Build-Env-Credential`
    - Host Key Verification Strategy: `Non Verifying Verification Strategy`
    - Availability: `Keep this agent online as much as possible`
- NODE PROPERTIES
    - Select `Environment variables`
        - Click `Add`
        - 1st Variable:
            - Name: `GRADLE_HOME`
            - Value: `/opt/gradle/gradle-6.8.3`
        - 2nd Variable:
            - Name: `PATH`
            - Value: `$GRADLE_HOME/bin:$PATH`

    - Click `SAVE`
- NOTE: Make sure the `Agent Status` shows `Agent successfully connected and online` on the Logs
- NOTE: Repeat the process for adding additional Nodes

## 10. Setup a CI Integration Between `GitHub` and `Jenkins`
### Navigate to your GitHub project repository
   - Open the repository
   - Click on the repository `Settings`
       - Click on `Webhooks`
       - Click `Add webhook`
           - Payload URL: http://JENKINS-PUBLIC-IP-ADDRESS/github-webhook/
           - Content type: `application/json`
           - Active: Confirm it is `Enable`
           - Click on `Add Webhook`

## 11. CREATE MAVEN PROJECT PIPELINE JOB

### 11.1. Create Maven Build, Test and Deploy Pipeline
###### Maven Build Job
- Click on `New Item`
    - Name: `Maven-Continuous-Integration-Pipeline`
    - Type: `Pipeline`
    - Click: `OK`
        - Select: `GitHub project`, Project url: `YOUR_MAVEN_PROJECT_REPOSITORY`
    - Check The Box: `GitHub hook trigger for GITScm polling`
    - Pipeline
      - Definition: Select `Git`, Repository URL: `YOUR_MAVEN_PROJECT_REPOSITORY`
    - Branches to build: `*/maven-sonarqube-nexus-jenkins`
    - Script Path: `Jenkinsfile`
    - `APPLY` and `SAVE`

## 12. CREATE GRADLE PROJECT PIPELINE JOB

### 12.1. Create Gradle Build, Test and Deploy Pipeline
###### Maven Build Job
- Click on `New Item`
    - Name: `Gradle-Continuous-Integration-Pipeline`
    - Type: `Pipeline`
    - Click: `OK`
        - Select: `GitHub project`, Project url: `YOUR_MAVEN_PROJECT_REPOSITORY`
    - Check The Box: `GitHub hook trigger for GITScm polling`
    - Pipeline
      - Definition: Select `Git`, Repository URL: `YOUR_MAVEN_PROJECT_REPOSITORY`
    - Branches to build: `*/gradle-sonarqube-nexus-jenkins`
    - Script Path: `Jenkinsfile`
    - `APPLY` and `SAVE`

## 13. TEST YOUR PIPELINE

