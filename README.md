# Jenkins-Maven-Sonar-Integration

                  Pipeline Declarative-Pipeline-maven:
https://www.jenkins.io/blog/2017/02/07/declarative-maven-project/


Jenkinsfile (Declarative Pipeline)
pipeline { 
    agent any  
    stages { 
        stage('Build') { 
            steps { 
               echo 'This is a minimal pipeline.' 
            }
        }
    }
}

•	All Declarative Pipelines start with a pipeline section.
•	Select where to run this Pipeline, in this case "any" agent, regardless of label.
•	Declarative automatically performs a checkout of source code on the agent, whereas Scripted Pipeline users must explicitly call checkout scm,
•	A Declarative Pipeline is defined as a series of stages.
•	Run the "Build" stage.
•	Each stage in a Declarative Pipeline runs a series of steps.
•	Run the echo step to print a message in the Console Output.

![image](https://user-images.githubusercontent.com/121241275/213898044-6eb020c9-0af6-4eee-9b5d-c8b8d28b04c5.png)

Maven Declarative Pipe line 
pipeline{
    agent any
    tools{
        maven "maven-3.8.6"
    }
    stages{
        stage("code checkout"){
            steps{
                git "https://github.com/cloudtechmasters/springboot-maven-course-micro-svc.git"
            }
        }
        stage("build the project"){
            steps{
                sh 'mvn clean package install'
            }
        }
    }
}

##################################################################
Gradle Declarative Pipeline

![image](https://user-images.githubusercontent.com/121241275/213898065-4e45a4b9-b4c8-43d3-aa39-83a3729c84bd.png)

pipeline{
    agent any
    tools{
        gradle "gradle-7.6"
    }
    stages{
        stage("git clone"){
            steps{
                git "https://github.com/cloudtechmasters/nov13-gradle-demo.git"
            }
        }
        stage("project build"){
            steps{
                sh 'gradle clean build'
            }
        }
    }
}

#############################################################################################

Integrate Jenkins with SonarQube for code quality Test:
-------------------------------------------------------

Sonar Server:
Java must be installed to run the sonar
sudo yum install java-11-amazon-corretto-headless -y

sudo hostnamectl set-hostname Sonar-Server 
sudo -i
google: download sonarqube 
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.8.0.63668.zip
Imp:
1>	Its not recommended to run the sonar with root user, we need to change the ownwership of sonar folder to normal user like ec2-user or we can create the dedicated user name like sonar 

Let download the sonar in /opt folder
Cd  /opt 
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.8.0.63668.zip
unzip sonarqube-9.8.0.63668
/tmp/sonarqube-9.8.0.63668/bin/linux-x86-64/sonar.sh start

Sonar port 9000 must be open in security group
Access the Sonar UI through browser using url: http://13.126.255.253:9000/
Default user and password is admin, admin
Step to integrate the SonarQube with Jenkins-
At Sonar Side:
Generate the token

![image](https://user-images.githubusercontent.com/121241275/213898100-27816ccc-5638-4769-9edb-599bad0a3628.png)

Generate the webhook: Administrator-->Configuration-->Webhooks
![image](https://user-images.githubusercontent.com/121241275/213898112-4eb02e11-8253-41b5-b36d-ea97917ea83a.png)
![image](https://user-images.githubusercontent.com/121241275/213898114-b0ad234d-e450-471f-8768-3e6126a3088c.png)

At Jenkins Side:

First add the SonarQube plugin 
Add the SonarQube token in Credential setting-

![image](https://user-images.githubusercontent.com/121241275/213898122-b39f62db-a4f7-4629-b33b-7346a3606ba1.png)

Now tell Jenkins where the Sonar is running under configuration system
Dashboard-->Manage Jenkins-->Configure System

![image](https://user-images.githubusercontent.com/121241275/213898136-c3282666-15ef-49ab-9b27-1588e1980a7b.png)

Sonar scanner will not be available by default we need to add sonar scanner plugins under manage plugins
Just Search in google- sonar Jenkins pipeline
https://www.jenkins.io/doc/pipeline/steps/sonar/
pipeline {
        agent none
        stages {
          stage("build & SonarQube analysis") {
            agent any
            steps {
              withSonarQubeEnv('My SonarQube Server') {
                sh 'mvn clean package sonar:sonar'
              }
            }
          }
          stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
        }
      }

![image](https://user-images.githubusercontent.com/121241275/213898148-761bb8ef-7259-4574-9d34-be41359b1d26.png)


pipeline{
    agent any
    tools{
        maven "maven-3.8.6"
    }
    stages{
        stage("git clone"){
            steps{
                git "https://github.com/cloudtechmasters/springboot-maven-course-micro-svc.git"
            }
        }
        stage("Build project"){
            steps{
                sh 'mvn clean package'
            }
            
        }
        stage("build & SonarQube analysis") {
            steps {
              withSonarQubeEnv('sonar-9.8') {
                sh 'mvn clean package sonar:sonar'
              }
            }
          }
    }
}



IMP: For all project we can use the generic step for sonar scan using sonar scanner,
We need to install the sonar scanner on Jenkins machine that means sonar scanner always run on Jenkins machine.
We need to mention the sonar url and token in project.properties for every project. 

Google- sonar maven scanner
On Jenkins Machine-
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.7.0.2747-linux.zip

once sonar scanner downloaded we need to inform sonar scanner where our sonar running and what is the token for sonar-

/tmp/sonar-scanner-4.7.0.2747-linux/conf
![image](https://user-images.githubusercontent.com/121241275/213898164-4762e8ba-e1cb-45b6-8e9a-67aef6d033fa.png)

Make the entry into sonar-scanner.properties file add the sonar url and token in properties file
![image](https://user-images.githubusercontent.com/121241275/213898170-cadfc5a2-62d9-426f-a8a8-d9abc151fe36.png)



Now we can change our pipeline, we need to run the scanner for sonar scan 
Add the sonar-scanner full path in sh command
        stage("analysis using scanner") {
            steps {
                sh '/tmp/sonar-scanner-4.7.0.2747-linux/bin/sonar-scanner'
            }
          }

Note: sonar scanner always see the sonar-scanner.properties file under root folder of every project

![image](https://user-images.githubusercontent.com/121241275/213898185-a90a8ec3-83e2-4dc4-aa65-30e277d695e6.png)

pipeline{
    agent any
    tools{
        maven "maven-3.8.6"
    }
    stages{
        stage("git clone"){
            steps{
                git "https://github.com/cloudtechmasters/springboot-maven-course-micro-svc.git"
            }
        }
        stage("Build project"){
            steps{
                sh 'mvn clean package'
            }
            
        }
        stage("sonar-9.8") {
            steps {
                sh '/tmp/sonar-scanner-4.7.0.2747-linux/bin/sonar-scanner'
            }
          }
    }
}

Now add the one more stage called quality gate in which we can mention the time, means if the sonar scan takes time so in that case pipeline will be stop after mentioned time.

![image](https://user-images.githubusercontent.com/121241275/213898196-2c8de780-8482-45ae-91ac-f29392a4aaed.png)

pipeline{
    agent any
    tools{
        maven "maven-3.8.6"
    }
    stages{
        stage("git clone"){
            steps{
                git "https://github.com/cloudtechmasters/springboot-maven-course-micro-svc.git"
            }
        }
        stage("Build project"){
            steps{
                sh 'mvn clean package'
            }
            
        }
        stage("analysis using scanner") {
            steps {
                withSonarQubeEnv(sonar-9.8') {
                sh '/tmp/sonar-scanner-4.7.0.2747-linux/bin/sonar-scanner'
              }
            }
          }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
          }

    }
}
