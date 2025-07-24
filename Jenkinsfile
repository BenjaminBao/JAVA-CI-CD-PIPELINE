def COMMIT
def BRANCH_NAME
def GIT_BRANCH
pipeline
{
 agent any
 environment
 {
     AWS_ACCOUNT_ID="434000704561"
     AWS_DEFAULT_REGION="ap-southeast-1" 
     IMAGE_REPO_NAME="devsecops-images"
     REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
     GIT_REPO  = "https://github.com/dev1git/maven-web-application.git"
     NEXUS_URL = "54.169.118.38:8081"
     
 }
 tools
 {
      maven '3.9.11'
      'SonarScanner 4.0'
 }   

 options 
 {
  buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '4', daysToKeepStr: '', numToKeepStr: '4')
  timestamps()
}
 stages
 {
     stage('Code checkout')
     {
         steps
         {
             script
             {
                 checkout([$class: 'GitSCM', branches: [[name: '*/dev']], extensions: [], userRemoteConfigs: [[url: '${GIT_REPO}']]])
                 COMMIT = sh (script: "git rev-parse --short=10 HEAD", returnStdout: true).trim()                   

             }
             
         }
     }
     stage('Build')
     {
         steps
         {
             sh "mvn clean package"
         }
     }
     stage('Execute Sonarqube Report')
     {
         steps
         {
            withSonarQubeEnv('sonarqube') 
             {
                sh "mvn sonar:sonar"
             }  
         }
     }
     stage('Quality Gate Check')
     {
         steps
         {
             timeout(time: 1, unit: 'HOURS') 
             {
                waitForQualityGate abortPipeline: true, credentialsId: 'sonar-token'
            }
         }
     }
     
     stage('Nexus Upload')
     {
         steps
         {
             script
             {
                 def readPom = readMavenPom file: 'pom.xml'
                 def nexusrepo = readPom.version.endsWith("SNAPSHOT") ? "maven-snapshot" : "maven-release"
                 nexusArtifactUploader artifacts: 
                 [
                     [
                         artifactId: "${readPom.artifactId}",
                         classifier: '', 
                         file: "target/${readPom.artifactId}-${readPom.version}.war", 
                         type: 'war'
                     ]
                ], 
                         credentialsId: 'nexus', 
                         groupId: "${readPom.groupId}", 
                         nexusUrl: ${NEXUS_URL}, 
                         nexusVersion: 'nexus3', 
                         protocol: 'http', 
                         repository: "${nexusrepo}", 
                         version: "${readPom.version}"

             }
         }
     }

     /*
     stage('Login to AWS ECR')
     {
         steps
         {
             script
             {
                 sh "/usr/local/bin/aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
             }
         }
     }
     stage('Building Docker Image')
     {
         steps
         {
             script
             {
              sh "docker build . -t ${REPOSITORY_URI}:mavenwebapp-${COMMIT}"
             }
         }
     }
     stage('Pushing Docker image into ECR')
     {
         steps
         {
             script
             {
                 sh "docker push ${REPOSITORY_URI}:mavenwebapp-${COMMIT}"
             }
         }

     }
     stage('Update image in K8s manifest file')
     {
         steps
         {
             
                 sh """#!/bin/bash
                 sed -i 's/VERSION/$COMMIT/g' deployment.yaml
                 """
             }
         }
     
     stage('Deploy to K8s cluster')
     {
         steps
         {
             
             sh '/usr/local/bin/kubectl apply -f deployment.yaml --record=true'
             sh """#!/bin/bash
             sed -i 's/$COMMIT/VERSION/g' deployment.yaml
             """

         }
     }
     */
 }

 post
 {
     always
     {
         cleanWs()
     }

     /*
     success
     {
        slackSend channel: 'build-notifications',color: 'good', message: "started  JOB : ${env.JOB_NAME}  with BUILD NUMBER : ${env.BUILD_NUMBER}  BUILD_STATUS: - ${currentBuild.currentResult} To view the dashboard (<${env.BUILD_URL}|Open>)"
        emailext attachLog: true, body: '''BUILD IS SUCCESSFULL - $PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS:
 
        Check console output at $BUILD_URL to view the results.
 
        Regards,
 
        Nithin John George
        ''', compressLog: true, replyTo: 'njdevops321@gmail.com', 
        subject: '$PROJECT_NAME - $BUILD_NUMBER - $BUILD_STATUS', to: 'njdevops321@gmail.com'
     }
     failure
     {
         slackSend channel: 'build-notifications',color: 'danger', message: "started  JOB : ${env.JOB_NAME}  with BUILD NUMBER : ${env.BUILD_NUMBER}  BUILD_STATUS: - ${currentBuild.currentResult} To view the dashboard (<${env.BUILD_URL}|Open>)"
         emailext attachLog: true, body: '''BUILD IS FAILED - $PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS:
 
        Check console output at $BUILD_URL to view the results.
 
        Regards,
 
        Nithin John George
        ''', compressLog: true, replyTo: 'njdevops321@gmail.com', 
        subject: '$PROJECT_NAME - $BUILD_NUMBER - $BUILD_STATUS', to: 'njdevops321@gmail.com'
     }
     */
 }

}
