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
        sh 'docker run gesellix/trufflehog --json https://github.com/NTD1810/webapp.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
    
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
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
          sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@10.10.10.42:/home/ubuntu/apache-tomcat-8.5.89/webapps/webapp.war'
          }      
       }       
    }
    
    stage ('DAST') {
      steps {
        sshagent(['zap']) {
          sh 'ssh -o  StrictHostKeyChecking=no ubuntu@10.10.10.41 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://10.10.10.42:8080/webapp/" || true'
        }
      }
    }
  }
}
