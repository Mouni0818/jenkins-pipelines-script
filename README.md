pipeline{
    agent any
    stages{
        stage('checkout-master'){
            steps{
              git credentialsId: 'mouni-git', url: 'https://github.com/Mouni0818/maven_project.git'
            }
        }  
        stage('Build the master code'){
            steps{
              sh "mvn package -f pom.xml"
            }
        } 
        stage('Deployment'){
            steps{
              deploy adapters: [tomcat9(credentialsId: 'tomcat_deploy', path: '', url: 'http://34.125.125.182:8080/')], contextPath: null, war: 'target/DEVOPS-GIT.war'
            }
        }  
        stage ('Approvals to move war to nexus') {
           steps {
           echo "Taking approval from  Manager for artifacts to NEXUSREPO"
           timeout(time: 7, unit: 'DAYS') {
            input message: 'Do you want to move war to NEXUS REPO?', submitter: 'kmouni0818'
           }
           }
        }
        stage('nexus upload'){
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'DEVOPS-GIT', classifier: '', file: 'target/DEVOPS-GIT.war', type: 'war']], credentialsId: 'nexus', groupId: 'com.devops', nexusUrl: '34.125.41.124:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
            }
        } 
        stage('checkout from branch'){
             steps{
                 git branch: 'Release_0.17', credentialsId: 'mouni-git', url: 'https://github.com/Mouni0818/maven_project.git'
             }
        }
        stage('Build branch code'){
            steps{
              sh "mvn package -f pom.xml"
            }
        } 
        stage ('Approvals to deploy') {
           steps {
           echo "Taking approval from  Manager for artifacts deploy to PRE-PROD ENV"
           timeout(time: 7, unit: 'DAYS') {
            input message: 'Do you want to deploy war in PRE-PROD ENV ?', submitter: 'kmouni0818'
           }
           }
        }
        stage('DEPLOY to PRE-PROD env'){
            steps{
                deploy adapters: [tomcat9(credentialsId: 'tomcat_deploy', path: '', url: 'http://34.125.125.182:8080/')], contextPath: null, war: 'target/DEVOPS-GIT.war'
            }
        }
        stage ('Approvals to move release war to nexus') {
           steps {
           echo "Taking approval from  Manager for artifacts to NEXUSREPO"
           timeout(time: 7, unit: 'DAYS') {
            input message: 'Do you want to move war to NEXUS REPO?', submitter: 'kmouni0818'
           }
           }
        }
        stage('nexus upload from branch'){
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'DEVOPS-GIT', classifier: '', file: 'target/DEVOPS-GIT.war', type: 'war']], credentialsId: 'nexus', groupId: 'com.devops', nexusUrl: '34.125.41.124:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
                
            }
        }
        
    }
 }
 
