# TOOLS INSTALLATION AND CONFIGURATION 

**Order of Setup Server Set-up**
- Jenkins-Controller/Master Server
- Nexus Server
- SonarQube Server
- Maven Server
- Gradle Server


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

**Note**: 
- ssh into JenkinsSever, `java -version` to ensure the teh server is set to `java-openjdk11`
- Else, run Run Java alternative Commands: `sudo alternatives --config java`    # the raw but not to run: ` sudo update-alternatives --config javac`
- Repeat the above on Maven and 


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


## 5. UPDATE YOUR MAVEN CONFIGURATIONS FILES

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
  **Note**: This is especially needed when using declarative approach with jenkinsfile, else while testing out with imperative the `SonarQube Scanner Snippet` must be provided at the Jenkins Server UI when configuring teh jobs/pipeline

## 6. UPDATE YOUR GRADLE CONFIGURATIONS

### A) Update The Nexus PrivateIP Address & Repository Name For Gradle Project In The `build.gradle` File
 - Switch to the <'gradle-sonarqube-nexus-jenkins'>
 - Line: '57'

### B) Add The SonarQube Scanner Configurations API In The Gradle 'build.gradle' File
- Line: '37' - '44'

## 7. Set-up Maven Server: "Maven-Agent-JDK11"
- **name**: Maven-Agent-JDK11
- **machine type**: t2.mmicro or t2.medium for more resources
- **image/ami**: Amazon Linux 2
- **Key pair**: Select or Create
- **Security group ports**:  SSH port 22, Allow traffic from anywhere 0.0.0.0/0
- **user-data-in-the-advanced-details**: Use "maven-install-config.sh" in  the config folder of this repo and branch  https://github.com/EngineerChris/jenkins-controller-agent-config/blob/main/archive/maven-install.md  #update the settings.xml github path if not done already.

### 7.1 Check Settings
- ssh into Maven-Agent-Server: ssh -i kp.pem ec2-user@ip
- check the maven version and details: mvn -v, which maven  
- Check Java Version to ensure it is JDK11  to ensure unform version across all tools server 
   - `java  -version `
   - **If the Version is different**:  Follow the below steps:
      - `ssh jenkinsmaster@ipAddressof MAVEN INSTANCE ` # Logging into Maven Server as Jenkinsmaster user.
      - ` sudo su ec2-user` # Swicth to ec2-user
      - pwd (result = /home/ec2-user)
      - Run Java alternative Commands: `sudo alternatives --config java`    #the raw but not to run: $ sudo update-alternatives --config javac
      - pass the number that corresponds to Java-11-open...: e.g `1`
      - `java  -version ` again to confirm
  - **Test that Maven can be executed within this env**: mvn validate (Build failed error is ok at this point, no project is being built)


## 8. Set-up Gradle Server: "Gradle-Agent-JDK11"
- **name**: Gradle-Agent-JDK11
- **machine type**: t2.mmicro  t2.medium for more resources
- **image/ami**: Ubuntu 22.04
- **Key pair**: Select or Create
- **Security group ports**:  22, Allow traffic from anywhere 0.0.0.0/0
- **user-data-in-the-advanced-details**: https://github.com/EngineerChris/jenkins-controller-agent-config/blob/main/archive/gradle-install.md
- ssh into M=Gradle-Agent-Server: ssh -i kp.pem ec2-user@ip
  - check the gradle version and details: gradle, which gradle
- **Test that Gradle can be executed within this env**: mvn validate (Build failed error is ok at this point, no project is being built )



## 9. CHECK IF THE USER "JENKINSMASTER" IS CREATED IN BOTH MAVEN AND GRADLE SERVERS
- On the normal Local Terminal Status (Not connected to any EC2)
- <SSH jenkinsmaster@MavenInstanceIP> : To confirm if the created user jenkinmaster is there in that @server with the iP

```bash
<SSH jenkinsmaster@MavenInstanceIP> : To confirm if the created user jenkinmaster is there in that @server with the iP
<whoami>
 <sudo yum update> Test if the user(masters) has escalated privilieges 
sudo ls /opt
ls -al /opt  #you will notice that opt belongs to jenkinsmaster
**Repeate for Gradle**
gradle build

```

# JENKINS CONFIGURATION 

## 1.0 CONFIGURE CONTROLLER AND AGENT
- Click on "Manage Jenkins" >> Click "Nodes and Cloud" >> Click "New Node"
- Click `New Node`
    - Node name: `Maven-Build-Env` 
    - Type: `Permanent Agent` >> Click `CREATE`

#### 1.1. Configure "Maven-Build-Env"
- Name:                  `Maven-Build-Env`
- Number of Executors:   `5` (for example, maximum jobs to execute at a time)
- Remote root directory: `/opt/maven-builds`# workspace that jenkins will use to store all  build executable generated by jenkins in the maven build env.
- Labels:                `Maven-Build-Env`
- Usage:                 `Use this node as much as possible`
- Launch method:         `Launch agents via SSH`
    - Host:   `Provide IP of Maven-Build-Server`
    - Credentials:
        - Click on `Add / Jenkins` and Select `Username and Password`
        - Username: `jenkinsmaster` #created during creating in maven install script
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
            - Value: `/usr/share/apache-maven`   # From the [.bash_profile link](https://raw.githubusercontent.com/awanmbandi/realworld-cicd-pipeline-project/jenkins-master-client-config/.bash_profile)
        - 2nd Variable:
            - Name: `PATH`
            - Value: `$MAVEN_HOME/bin:$PATH`

    - Click `SAVE`
- NOTE: Make sure the `Agent Status` shows `Agent successfully connected and online` on the Logs
- Go to Nodes > Maven-Build-Env > Status > Relaunch Agent
- NOTE: Repeat the process for adding additional Nodes

#### 1.2. Configure "Gradle-Build-Env"
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
- Go to Nodes > Gradle-Build-Env > Status > Click "Relaunch Agent" [ Jenkins will try to connect to the Agent, to see if it can execute agent.jar file]
- NOTE: Repeat the process for adding additional Nodes

## 2.0. Setup a CI Integration Between `GitHub` and `Jenkins`
### Navigate to your GitHub project repository
   - Open the repository
   - Click on the repository `Settings`
       - Click on `Webhooks`
       - Click `Add webhook`
           - Payload URL: http://JENKINS-PUBLIC-IP-ADDRESS/github-webhook/
           - Content type: `application/json`
           - Active: Confirm it is `Enable`
           - Click on `Add Webhook`


## 3.0. CREATE MAVEN PROJECT PIPELINE JOB

**Important: Install "Delivery Pipeline Plugins"**
Since we are leveraging Imperative Approach (through "Build other Jobs") to execute the jobs in this project,  installing  "Delivery Pipeline Plugins" is require for nice visuals.
  - Delivery Pipeline Plugins enhance Imperative Approach by providing essential visualization, monitoring, and management capabilities. These plugins turn the scripted pipeline into a more manageable, understandable, and efficient process, which is critical for maintaining and scaling your continuous delivery efforts..
  - DashBoard -> Manage Jenkins -> Plugins -> search for "Delivery Pipeline" -> click "Install without restart

### 3.1 Create Maven Build, Test and Deploy Pipeline/job
##### 3.1.1 Maven Build Job
- Click on `New Item`
    - Name: `Maven-Continuous-Integration-Pipeline-Build`
    - Type: `Pipeline` #one for Build, another for test and lastly another for Deploy
    - Click: `OK`
        - Select: `GitHub project`, Project url: `YOUR_MAVEN_PROJECT_REPOSITORY - GITHUB LINK`
    - Check The Box: `GitHub hook trigger for GITScm polling`
    - Pipeline / Source Code Management
      - Definition: Select `Git`, Repository URL: `YOUR_MAVEN_PROJECT_REPOSITORY`
    - Branches to build: `*/maven-sonarqube-nexus-jenkins`
    - Script Path: `Jenkinsfile` # for later 
    - Restrict where this project can be run: `Maven-Build-Env`
    - Build Steps: Add build step > Provide the Build Command "mvn package" or "mvn clean package"
    - `APPLY` and `SAVE`

##### 3.1.2 Maven Test Job
*Pre-requisite*:  Create token on SonarQube Server Copy the SonarQube Analysis code snipet (starts with `mvn sonar:sonar`) and save somewhere ready for use. This code always get created with SQ PubIP so ensure to replace the SQ publicIP with Private

- Click on `New Item`
    - Name: `Maven-Continuous-Integration-Pipeline-SonarQube-Test`
    - Type: `Pipeline` 
    - Click: `OK`
        - Select: `GitHub project`, Project url: `YOUR_MAVEN_PROJECT_REPOSITORY - GITHUB LINK`
    - Check The Box: `GitHub hook trigger for GITScm polling`
    - Pipeline
      - Definition: Select `Git`, Repository URL: `YOUR_MAVEN_PROJECT_REPOSITORY`
    - Branches to build: `*/maven-sonarqube-nexus-jenkins`   #the specicic branch
    - Script Path: `Jenkinsfile` #for later DECLARATIVE APPROACH
    - Restrict where this project can be run: `Maven-Build-Env`
    - Build Steps: `Add build step > Provide the SonarQube Analys Command from the Sonar Server`
      - paste the SOnarQube Analysis code snipet Command after ensuring to change the SonarQube Public IP to PrivateIP
      - `NOTE`: BUILD STEPS will no be displayed once `Jenkinsfile` is chosen at the level of script path. Build steps are are already defined in the Jenkins file
    - `APPLY` and `SAVE`

##### 3.1.3 Maven Upload Job
*Pre-requisite*:  Create token on SonarQube Server Copy the SonarQube Analysis code snipet (starts with `mvn sonar:sonar`) and save somewhere ready for use. This code always get created with SQ PubIP so ensure to replace the SQ publicIP with Private

- Click on `New Item`
    - Name: `Maven-Continuous-Integration-Pipeline-Nexux-Upload`
    - Type: `Pipeline` 
    - Click: `OK`
        - Select: `GitHub project`, Project url: `YOUR_CONTOLLER-AGENT_PROJECT_REPOSITORY - GITHUB LINK`
    - Check The Box: `GitHub hook trigger for GITScm polling`
    - Restrict where this project can be run: `Maven-Build-Env`
    - Pipeline / Source Code Managemet=nt
      - Definition: Select `Git`, Repository URL: `YOUR_MAVEN_PROJECT_REPOSITORY`
    - Branches to build: `*/maven-sonarqube-nexus-jenkins`   #the specicic branch
    - Script Path: `Jenkinsfile`
    - Build Steps: Add build step > Provide the Build Command `mvn deploy` # deploy to nexus
    - `APPLY` and `SAVE`

### 3.2 Connect all three jobs Together to Have Continous Integration Pipeline - Configure Post-build-Action (Build other pojects)
To Integrate the jobs/pipeline to one another such that when one jobs finishes, it will triger the next.
- **Integrate first job with the second**:  Dashboad > `Maven-Continuous-Integration-Pipeline-Build` > Configuration > Build Steps > Post-build-Actions > Add post-build-action (dropdown) > Build other pojects  > choose next job to build --> `Maven-Continuous-Integration-Pipeline-SonarQube-Test` > Trigger only if build is stable > SAVE'

- **Integrate second job(test) with the third job(upload)**: Dashboad > `Maven-Continuous-Integration-Pipeline-SonarQube-Test` > Configuration > Build Steps > Post-build-Actions > Add post-build-action (dropdown) > Build other pojects  > choose next job to build -->  `Maven-Continuous-Integration-Pipeline-Nexux-Upload` > Trigger only if build is stable > SAVE'

### 3.3 Create pipeline view for the three jobs
Dashboad > + (new view) > Name=`Maven-Continuous-Integration-Pipeline-view > Delivery Pipeline View` > create > description(pass same as given to name) > number of pipeline instances = (3) since we have 3 jobs, > Component > Add > Component Name = `Maven-Continuous-Integration-Pipeline-view` > initial job ( i.e the first job)=`Maven-Continuous-Integration-Pipeline-Build`> APPLY > SAVE    

**Check**
All 3 jobs related to Node "Maven-Build-Env should be showing under it: 
- "Dashboard > Manage Jenkins > Nodes > Maven-Build-Env"

### 3.4 Test Maven Project Pipelines/Jobs
    - **Pre-requisite** Confirm that all teh Servers (SQ, Nexus, Maven, Jenkins ec2) are up. Ensure right configurations are provided in the the Settings.XML and POM.XML. In Ensure private IPs and Repo Name to publish teh artifacts to are well defined within the configuration file POM.XML and Settings.XML where required.

**Build Now To Test**: Dashboard (or go from from Nodes&CLouds under Manage Jenkins) > Click on first job `Maven-Continuous-Integration-Pipeline-Build` > Build Now
- Check SonarQube Server for test result 
- Check Nexus Server for test result (box > browse > maven-java-webapp-repo > JavaWebApp) 
- To view Pipeline Stages: Click on `Maven-Continuous-Integration-Pipeline-view`

## 4.0 CREATE GRADLE PROJECT PIPELINE JOB

### 4.1 Create Gradle Build, Test and Deploy Pipeline
##### 4.1.1 Gradle Build Job
- Click on `New Item`
    - Name: `Gradle-Continuous-Integration-Pipeline-Build`
    - Type: `Pipeline`
    - Click: `OK`
      - Select: `GitHub project`, Project/Repo url: `YOUR_CONTOLLER-AGENT_PROJECT_REPOSITORY LINK`
    - Restrict where this Project Can Run: ` Gradle-Build-Env`
    - Source Code Management / Pipeline :
      - Select `Git' >  'Repository' : ` Repository URL: `YOUR_CONTOLLER-AGENT_PROJECT_REPOSITORY LINK`
    - Branches to build: `*/gradle-sonarqube-nexus-jenkins`
    - Build Steps: 
      - Execute Sehell 
      - Command : `gradle clean build `
    - `APPLY` and `SAVE`

     **BUILD TRIGGER WEB-HOOK TO BE SET UP LATER **
    - Check The Box: `GitHub hook trigger for GIT Scm polling`
    - Script Path: `Jenkinsfile`
    
###### 4.1.2 Gradle Test Job
- Click on `New Item`
    - Name: `Gradle-Continuous-Integration-Pipeline-SonarQube-Test`
    - Type: `Pipeline`
    - Click: `OK`
      - Select: `GitHub project`, Project/Repo url: `YOUR_CONTOLLER-AGENT_PROJECT_REPOSITORY LINK`
    - Restrict where this Project Can Run: ` Gradle-Build-Env`
    - Source Code Management / Pipeline :
      - Select `Git' >  'Repository' : ` Repository URL: `YOUR_CONTOLLER-AGENT_PROJECT_REPOSITORY LINK`
    - Branches to build: `*/gradle-sonarqube-nexus-jenkins` #copy branch name from Git
    - Build Steps: 
      - Execute Sehell 
      - Command : `gradle sonarqube ` # the SonarQube Analysis Code from SonarQube Code has been passed within `build.gradle file` so we don't need to provide it within the Jenkins Server UI as we did when configuring jobs for Maven-based Project. `build.gradle` is as `POM.XL` for maven
    - `APPLY` and `SAVE`

     **BUILD TRIGGER WEB-HOOK TO BE SET UP LATER **
    - Check The Box: `GitHub hook trigger for GIT Scm polling`
    - Script Path: `Jenkinsfile`


###### 4.1.3 Gradle Upload Job
- Click on `New Item`
    - Name: `Gradle-Continuous-Integration-Pipeline-Nexus-Upload`
    - Type: `Pipeline`
    - Click: `OK`
      - Select: `GitHub project`, Project/Repo url: `YOUR_CONTOLLER-AGENT_PROJECT_REPOSITORY LINK`
    - Restrict where this Project Can Run: ` Gradle-Build-Env` label
    - Source Code Management / Pipeline :
      - Select `Git' >  'Repository' : ` Repository URL: `YOUR_CONTOLLER-AGENT_PROJECT_REPOSITORY LINK`
    - Branches to build: `*/gradle-sonarqube-nexus-jenkins` #copy branch name from Git
    - Build Steps: 
      - Execute Sehell 
      - Command : `gradle publish `  # Details and information of where to published has been defined in the `build.gradle`  file line 38-56
    - `APPLY` and `SAVE`

    **BUILD TRIGGER WEB-HOOK TO BE SET UP LATER**
    - Check The Box: `GitHub hook trigger for GIT Scm polling`
    - Script Path: `Jenkinsfile`

### 4.2 Connect all three jobs Together to Have Continous Integration Pipeline  for the Imperative Jobs- Configure Post-build-Action (Build other pojects)
To Integrate the jobs/pipeline to one another such that when one jobs finishes, it will triger the next.

- **Integrate first job with the second**:  Dashboad > `Gradle-Continuous-Integration-Pipeline-Build` > Configuration > Build Steps > Post-build-Actions > Add post-build-action (dropdown) > Build other pojects  > choose next job to build --> `Gradle-Continuous-Integration-Pipeline-SonarQube-Test` > Trigger only if build is stable > SAVE'

- **Integrate second job(test) with the third job(upload)**: Dashboad > `Gradle-Continuous-Integration-Pipeline-SonarQube-Test` > Configuration > Build Steps > Post-build-Actions > Add post-build-action (dropdown) > Build other pojects  > choose next job to build -->  `Gradle-Continuous-Integration-Pipeline-Nexux-Upload` > Trigger only if build is stable > SAVE'

### 4.3 Create pipeline view for the three jobs

- Dashboad > + (new view) > Name=`Gradle-Continuous-Integration-Pipeline-view > Delivery Pipeline View` > create > description(pass same as given to name) > number of pipeline instances = (3) since we have 3 jobs, > Component > Add > Component Name = Gradle-Continuous-Integration-Pipeline-view` > initial job ( i.e the first job)=`Gradle-Continuous-Integration-Pipeline-Build`> APPLY > SAVE    

**Check** All 3 jobs related to Node `Gradle-Build-Env` should be showing under the node.
- "Dashboard > Manage Jenkins > Nodes > Gradle-Build-Env"

### 4.4 Test Gradle Project Pipelines/Jobs
- **Pre-requisite**:  Confirm that all the Servers (SQ, Nexus, Gradle, Jenkins ec2 instances) are up and running. Ensure right configurations are provided in the the Settings.XML and POM.XML. In Ensure private IPs and Repo Name to publish the artifacts to are well defined within the configuration file `build.gradle` file.

- **Build Now To Test**: Dashboard( or go from from Node&CLouds under Manage Jenkins) > Click on first job `Gradle-Continuous-Integration-Pipeline-Build` > Build Now
- Check SonarQube Server for test result. `Project` > `Gradle-JavaWebApp`
- Check Nexus Server for test result (box > browse > gradle-java-webapp-repo > gradle-java-WebApp-repository) 
- To view Pipeline Stages: Click on `Gradle-Continuous-Integration-Pipeline-view`



## SETUP WEBHOOK FOR JENKINS AUTO-TRIGGER FROM GIT COMMIT
For BUILDING PROCESS to be auto-triggered whenever a new code chnage is is commited on Github

### 1. On Github*
Github> ProjectRepo > Settings > Webhook > Add webhook > http://JenkinsServerIP:8080/github-webhook/  > just to push event > Add webhook > Ensure you see Greencheck after refresh

### 2. On Jenkins Serve UI
Target build jobs (the first jobs) in each project and set  the `BuildTrigger` to `GitHub hook trigger for GITScm Polling`
**- Maven Project** : Dashboard >  `Maven-Continuous-Integration-Pipeline-Build`> Configure > BuildTrigger > GitHub hook trigger for GITScm Polling > Apply > Save
**- Gradle Project** : Dashboard >  `Gradle-Continuous-Integration-Pipeline-Build` > Configure > BuildTrigger > GitHub hook trigger for GITScm Polling > Apply > Save

### Test
**Maven Project**
Go to Local Environment ( not connected to ec2 ) > cd into the project repo (`jenkins-controller-agent-project`) > branch > maven-sonarqube-nexus-jenkins > commit any chnage (e.g go to `/src/main/java/com/example/demo/DemoApplication.java` to delete or add an unwanted space e.g btw line11 and 12) > commit to github
This should trigger Maven Project, specifically the job ` `Maven-Continuous-Integration-Pipeline-Build`
- Quickly go to Jenkins Server to see it running
- Nexus Check on Maven Project - You should see a new artifacts created.

**Gradle Project**
Go to Local Environment ( not connected to ec2 ) > cd into the project repo (`jenkins-controller-agent-project`) > git brnach > git checkout gradle-sonarqube-nexus-jenkins` to switch ` > make any chnage (e.g go to `/src/main/java/com/cloutechmaster/contoller/model/customer.java`  OR `/src/main/java/com/cloutechmaster/contoller/model/sampleWebApplication.java` to delete or add an unwanted line ) > commit to github

This should trigger Gradle Project, specifically the job ` `Gradle-Continuous-Integration-Pipeline-Build`
- Quickly go to Jenkins Server to see it running
- Nexus Check on Gradle Project - You should see a new artifacts created under box> Browse >gradle-java-webapp-repository 


# SET-UP DECLARATIVE ROUTE WITH JENKINSFILE 

- Script Path: `Jenkinsfile`



## NOTE 
### Some Usedata for Tools Configuration:
- jenkins-controller-agent-config: https://github.com/EngineerChris/jenkins-controller-agent-config/tree/main

### The approach for excuting the job:
1. **Imperative Approach** through "Build other Jobs" - chosen for this project
  - The imperative approach involves explicitly scripting the steps of your pipeline, often using tools like Jenkins' scripted pipelines 
  - Install DELIVERY PIPELINE PLUGINS for nice visuals when using using Imperative Jobs.
  - DashBoard -> Manage Jenkins -> Plugins -> search for "Delivery Pipeline" -> click "Install without restart"
2. **Declaritive Approach**: Create two jenkinsfile for the two environments (Maven and Gradle) and connect to the respective Github branches/repos. When using Jenkinfile, installing Delivery Pipeline Pluging is not needed as the the stages visual is preconfigured
