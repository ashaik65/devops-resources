######DevSecops Step by Step#########

https://github.com/ashaik65/kubernetes-devops-security

For Do this Activity We need Azure Account if anybody don't have an azure account any cloud provider is working we just need to create on Virtual Machine which has minimum configuration is 2 CPU Core and atlist 4GB RAM and HDD-60GB

Once The Virtual Machine has been created We Need to clone The Repo and get it to that Folder

root@ip-172-31-29-170:~# git clone https://github.com/sidd-harth/kubernetes-devops-security.git
Cloning into 'kubernetes-devops-security'...
remote: Enumerating objects: 95, done.
remote: Total 95 (delta 0), reused 0 (delta 0), pack-reused 95
Unpacking objects: 100% (95/95), done.

Go to this path 
root@ip-172-31-24-174:~/kubernetes-devops-security/setup/vm-install-script# ls
install-script.sh

after that fire this script  install-script.sh so it will install kubernetes, Docker, Jenkins 

root@ip-172-31-29-170:~/kubernetes-devops-security/setup/vm-install-script# bash install-script.sh 
.........----------------#################._.-.-INSTALL-.-._.#################----------------.........
Reading package lists... Done
Building dependency tree       
Reading state information... Done
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
0 upgraded, 1 newly installed, 0 to remove and 29 not upgraded.
Need to get 87.7 MB of archives.
After this operation, 91.3 MB of additional disk space will be used.
Get:1 https://pkg.jenkins.io/debian-stable binary/ jenkins 2.346.1 [87.7 MB]
Fetched 87.7 MB in 10s (8660 kB/s)                                                                                                                                               
Selecting previously unselected package jenkins.
(Reading database ... 81990 files and directories currently installed.)
Preparing to unpack .../jenkins_2.346.1_all.deb ...
Unpacking jenkins (2.346.1) ...
Setting up jenkins (2.346.1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/jenkins.service → /lib/systemd/system/jenkins.service.
Processing triggers for systemd (237-3ubuntu10.53) ...
Processing triggers for ureadahead (0.100.0-21) ...
Synchronizing state of jenkins.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable jenkins
.........----------------#################._.-.-COMPLETED-.-._#################---------........

After That check all resources are working or not like kubectl docker 

now look for jenkins whether its up and running using command 

root@ip-172-31-29-170:~# systemctl status jenkins
● jenkins.service - Jenkins Continuous Integration Server
   Loaded: loaded (/lib/systemd/system/jenkins.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2022-07-07 05:50:15 UTC; 1min 59s ago
 Main PID: 21591 (java)
    Tasks: 35 (limit: 4680)
   CGroup: /system.slice/jenkins.service
           └─21591 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=/var/cache/jenkins/war --httpPort=8080

Jul 07 05:50:05 ip-172-31-29-170 jenkins[21591]: Please use the following password to proceed to installation:
Jul 07 05:50:05 ip-172-31-29-170 jenkins[21591]: 9f9d6b14667d48ce916d248d188394fd
Jul 07 05:50:05 ip-172-31-29-170 jenkins[21591]: This may also be found at: /var/lib/jenkins/secrets/initialAdminPassword
Jul 07 05:50:05 ip-172-31-29-170 jenkins[21591]: *************************************************************
Jul 07 05:50:15 ip-172-31-29-170 jenkins[21591]: 2022-07-07 05:50:15.310+0000 [id=27]        INFO        jenkins.InitReactorRunner$1#onAttained: Completed initialization
Jul 07 05:50:15 ip-172-31-29-170 jenkins[21591]: 2022-07-07 05:50:15.333+0000 [id=20]        INFO        hudson.lifecycle.Lifecycle#onReady: Jenkins is fully up and running
Jul 07 05:50:15 ip-172-31-29-170 systemd[1]: Started Jenkins Continuous Integration Server.
Jul 07 05:50:15 ip-172-31-29-170 jenkins[21591]: 2022-07-07 05:50:15.458+0000 [id=43]        INFO        h.m.DownloadService$Downloadable#load: Obtained the updated data file for
Jul 07 05:50:15 ip-172-31-29-170 jenkins[21591]: 2022-07-07 05:50:15.462+0000 [id=43]        INFO        hudson.util.Retrier#start: Performed the action check updates server succ
Jul 07 05:50:15 ip-172-31-29-170 jenkins[21591]: 2022-07-07 05:50:15.465+0000 [id=43]        INFO        hudson.model.AsyncPeriodicWork#lambda$doRun$1: Finished Download metadata

Now access Jenkins in UI as its running on port 8080 
http://<external-ip/domain-name:8080>

Now Install Jenkins Plugins via script

root@ip-172-31-29-170:~/kubernetes-devops-security/setup/jenkins-plugins# bash plugin-installer.sh 
http://localhost:8080
22e19f8e300ab8841457dd71a05f3a55e7764996d2916ca76916e8373b320532
11bb5fb2c9808032bd4acb2c85a1b4a292
........Installing performance@3.18 ..
........Installing docker-workflow@1.26 ..
........Installing dependency-check-jenkins-plugin@5.1.1 ..
........Installing blueocean@1.24.7 ..
........Installing jacoco@3.2.0 ..
........Installing slack@2.4.8 ..
........Installing sonar@2.13.1 ..
........Installing pitmutation@1.0-18 ..

and also cross verify in jenkins UI Managed Jenkins-->Managed Plugins-->Update Center
as we check installer not update kubernetes plugin we need to go and install from UI 
kubernetes CLI plugin need to install.

Now we are run simple pipline for checking purpose

pipeline {
    agent any

    stages {
        stage('git version') {
            steps {
                sh 'git version'
            }
        }
        
        stage('maven version') {
            steps {
                sh 'mvn -v'
            }
        }
        
        stage('docker version') {
            steps {
                sh 'docker -v'
            }
        }
        
        stage('kubernetes version') {
            steps {
                sh 'kubectl version --short'
            }
        }
    }
}

and build the job but we see the pipline failed as all versions are showing but kubernetes server version showing forbideen error because kubernetes not having authentication with the Jenkins now we need to configure kubernetes with the jenkins we are use kubeconfig file for authentication

Once we configure credentials in managed credentials now we need to add parameters in jenkins file like this



pipeline {
    agent any

    stages {
        stage('git version') {
            steps {
                sh 'git version'
            }
        }
        
        stage('maven version') {
            steps {
                sh 'mvn -v'
            }
        }
        
        stage('docker version') {
            steps {
                sh 'docker -v'
            }
        }
        
        stage('kubernetes version') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                sh 'kubectl version --short'
            }
            }
        }
    }
}

###### Now we are integrate github with jenkins for Maven Build #######

Steps:- Go to Github ---> Go to Repository ----> Settings ----> Webhook ----> Add Webhook ---->
http://<jenkins-url/external-ip>:8080/github-webhook/ ----> just push events ----> Save

Once its done now we need create pipline for maven

but this time we will select git hub repo url insted of pipline script

simple-pipeline is 


pipeline {
  agent any
  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   
    }
}

and pom.xml contents are this we are mentioned contents here because of moving forward we will update the stuff in that file

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.1.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>

	<groupId>com.devsecops</groupId>
	<artifactId>numeric</artifactId>
	<version>0.0.1</version>
	<name>numeric</name>
	<description>Demo for DevSecOps</description>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
			
		</plugins>
	</build>

</project>

##### Now will do Unit tests in jenkins pipeline ####
for doing unit testing we are using JaCoco pluign which is used for java unit testing
now we are add one more step in our jenkins pipline

pipeline {
  agent any

  stages {
    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"
      }
    }
  }
}

now we see build is successful but it not showing exact details about how many lines of codes are tested which one are passed for viewing this all details we need to add JaCoco plugin details in pom.xml and also add some steps in jenkins

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.1.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>

	<groupId>com.devsecops</groupId>
	<artifactId>numeric</artifactId>
	<version>0.0.1</version>
	<name>numeric</name>
	<description>Demo for DevSecOps</description>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
			<!--                   Jacoco Plugin                   -->
               <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.5</version>
                <executions>
                 <execution>
                  <goals>
                   <goal>prepare-agent</goal>
                  </goals>
                 </execution>
                 <execution>
                  <id>report</id>
                  <phase>test</phase>
                  <goals>
                   <goal>report</goal>
                  </goals>
                 </execution>
                 </executions>
                 </plugin>
                </plugins>
	</build>

</project>

add JaCoco plugin in pom.xml now add stage in jenkins as well

pipeline {
  agent any

  stages {
    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }
  }
}

Now build will trigger once we add stage in Jenkinsfile check the coverage report in jenkins console it will give it full report for us

##### Build the Docker Image and push it to the repo using Jenkins pipline #######

Now we are adding some more stages in Jenkins pipelie for order to push images in docker hub

pipeline {
  agent any

  stages {

    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }

    stage('Docker Build and Push') {
      steps {
          sh 'printenv'
          sh 'docker build -t ashaik65/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push ashaik65/numeric-app:""$GIT_COMMIT""'
        }
      }
  }
}

After commit this changes pipline has been failed because of docker socket not has permission and also jenkins dont have an permission to push images in docker so we need to configure docker credentials as well.

Add permission to chmod 777 /var/run/docker.sock
Add docker credentials in managed jenkins ----> Managed credentials

One more thing need to remember here for docker passoword do not use normal passoword insted create token in dockerhub account and use as password

pipeline {
  agent any

  stages {

    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }

    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t ashaik65/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push ashaik65/numeric-app:""$GIT_COMMIT""'
        }
      }
    }
  }
}

Now we can see the image has been pushed to docker hub now atlast we try to deploy those image on kubernetes cluster
we can also check on terminal using <docker images> command 

##### Deploy docker image on kubernetes using Jenkins Pipline #######

## Deploy Node-Service in Kubernetes Default Namespace

kubectl -n default create deploy node-app --image siddharth67/node-service:v1 
kubectl -n default expose deploy node-app --name node-service --port 5000


## Within /src/main/java/com/devsecops/NumericController.java  
## use node-service:5000/plusone as baseURL and 
## comment the localhost:5000

private static final String baseURL = "http://node-service:5000/plusone";
//private static final String baseURL = "http://localhost:5000/plusone";

pipeline {
  agent any

  stages {

    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and JaCoCo') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }

    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t ashaik65/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push ashaik65/numeric-app:""$GIT_COMMIT""'
        }
      }
    }

    stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#ashaik65/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
    
  }

}

Now check pod and service are deploy now try to access using public ip or domain name 

http://<external-ip>:NodePort
for example http://54.91.120.163:31974/ but we can also check because or app contain increment values 

http://54.91.120.163:31974/increment/50  its show output also spring boot application shows 
http://54.91.120.163:31974/compare/44 its shows output


