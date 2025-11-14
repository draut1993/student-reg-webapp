@Library('JenkinsSharedLibrary') _
pipeline {
    
    agent any
    
    triggers {
       githubPush()
     }

    options {
      buildDiscarder logRotator(numToKeepStr: '5')
      timeout(time: 10, unit: 'MINUTES')
      disableConcurrentBuilds()
    }
    
    environment {
        EMAIL = "draut3078@gmail.com"
        SONARQUBE_URL = "http://98.93.190.195:9000/"
        SONAR_QUBE_TOKEN = credentials('sonarToken')
        TOMCAT_SERVER_IP = "98.84.142.255"
    }
    
    tools {
        maven 'Maven_build'
    }

    stages {


       stage("Maven Clean Package"){
           steps {
               sh "mvn clean package"
           }
       }
       
      stage("Sonar Scan"){
           steps {
            sh "mvn sonar:sonar -Dsonar.host.url=${SONARQUBE_URL} -Dsonar.token=${SONAR_QUBE_TOKEN}"
          }
      }
      
      stage("Upload War To Nexus"){
          steps {
              sh "mvn clean deploy"
          }
      }
      
       stage("Deployt To Dev Server") {
    
        steps{
    
            sshagent(['Tomcat_server_SSH_Agent']) {
                sh """
                     ssh -o  StrictHostKeyChecking=no ec2-user@${TOMCAT_SERVER_IP} 'sudo /home/ec2-user/apache-tomcat-9.0.110/bin/shutdown.sh'
                     echo Stoping the Tomcat Process
                     sleep 30
                     scp -o  StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@${TOMCAT_SERVER_IP}:/home/ec2-user/apache-tomcat-9.0.110/webapps/student-reg-webapp.war
                     echo Copying the War file to Tomcat Server
                     ssh -o  StrictHostKeyChecking=no ec2-user@${TOMCAT_SERVER_IP} 'sudo /home/ec2-user/apache-tomcat-9.0.110/bin/startup.sh'
                     echo Strating the Tomcat process
                   """
            }
        }
      }
      
       
   }
   
   post {
        always {
            EmailNotification(currentBuild.currentResult, EMAIL)
        }
    }
}