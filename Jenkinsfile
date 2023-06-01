pipeline{

agent any

tools{
maven 'Maven 3.9.1'
}
stages{
   stage('CheckoutCode') {
   steps{  
   sendSlackNotifications ('STARTED')
   git branch: 'dev', credentialsId: '664e89b6-0792-4126-8a14-34910e75472c', url: 'https://github.com/hari-software-org/maven-web-application.git'
   }
   }
   stage('Build') {
   steps{
   sh "mvn clean package"
   }
   }
   stage('SonarQubeReport'){
   steps{
   sh "mvn clean sonar:sonar"
   }
   }
   stage('UploadArtifactstoNexus'){
   steps{
   sh "mvn clean deploy"
   }
   }
   stage('DeployApplication'){
   steps{
   sshagent(['7f80c562-8151-4e83-949f-65a30885f9a1']) {
   sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@172.31.81.69:/opt/apache-tomcat-9.0.75/webapps"
   }
   }
   }
   
}//stages closing
post {
  success {
  sendSlackNotifications (currentBuild.result)
  }
  failure {
   sendSlackNotifications (currentBuild.result)
  }
}
}//pipeline closing

def sendSlackNotifications(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }
  // Send notifications
  slackSend (color: colorCode, message: summary, channel: '#citibank')
}
