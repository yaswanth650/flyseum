 pipeline {
  agent any 
  tools {
    maven 'Maven'
  }
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
    stage ('Check-Git-Secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'docker run gesellix/trufflehog --json https://github.com/yaswanth650/flyseum.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh 'mvn package'
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    }
   stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }
   stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@43.204.30.68:/prod/apache-tomcat-10.1.30/webapps/flyseum.war'
              }      
           }       
    }
    stage ('DAST') {
      steps {
        sshagent(['owasp-zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ubuntu@52.66.247.221 "docker run -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t https://43.204.30.68:8443/flyseum/" || true'
        }
      }
    }
  }
}
