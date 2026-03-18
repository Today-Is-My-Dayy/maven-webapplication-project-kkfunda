//This is scripted-way pipeline 
node
{
   def mavenHome = tool name: "maven-3.9.14"
   echo "git branch Name: ${env.BRANCH_NAME}"
   echo "build number: ${env.BUILD_NUMBER}"
  try
  {


  stage('checkout')
  {
    git branch: 'dev', url: 'https://github.com/kkdevopsb8/maven-webapplication-project-kkfunda.git'
  }

  stage('compile')
  {
    sh "${mavenHome}/bin/mvn compile"
  }

  stage('BUILD')
  {
    sh "${mavenHome}/bin/mvn clean package"
  }

 /* stage('SQ REPORT')
  {
    sh "${mavenHome}/bin/mvn sonar:sonar"
  } */

  stage('Deploy to Nexus')
  {
    sh "${mavenHome}/bin/mvn deploy"
  }



  stage('Deploy to Tomcat') {
    if (currentBuild.currentResult == 'SUCCESS') {
        withCredentials([usernamePassword(
            credentialsId: 'tomcat-cread-1',
            usernameVariable: 'TOMCAT_USER',
            passwordVariable: 'TOMCAT_PASS'
        )]) {
            sh '''
            curl -u "$TOMCAT_USER:$TOMCAT_PASS" \
            --upload-file /var/lib/jenkins/workspace/jio-dev-pipeline/target/maven-web-application.war \
            "http://43.205.230.133:8080/manager/text/deploy?path=/maven-web-application&update=true"
            '''
        }
    } else {
        echo "Build failed or unstable. Skipping deployment."
    }
  }

  } //try block  ending
   //try block end
   catch (e) {
   
       currentBuild.result = "FAILED"

  } finally {
    // Success or failure, always send notifications
    notifyBuild(currentBuild.result)       //function calling
  }

} //node ending


def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESS') {
    color = 'GREEN'
    colorCode = '#278EF5'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary, channel: '#kkdevops-dev')
  
}
