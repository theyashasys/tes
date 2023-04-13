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
       sh 'docker run gesellix/trufflehog --json https://github.com/theyashasys/chilli.git > trufflehog'
        sh 'cat trufflehog'
     }
    }
    
  stage ('Source Composition Analysis') {
    steps {
         sh 'rm owasp* || true'
         dependencyCheck additionalArguments: '--format HTML', odcInstallation: 'owasp'
         sh ' mvn org.owasp:dependency-check-maven:check'
        // sh 'wget "https://raw.githubusercontent.com/cehkunal/webapp/master/owasp-dependency-check.sh" '
         //sh 'chmod -R 777 owasp-dependency-check.sh'
         //sh 'bash owasp-dependency-check.sh'
         //sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
        
      }
    }
    
    stage ('JAVA') {
    steps{
    sh 'java -version' 
    }
  }
    
    stage ('SAST') {
      steps {
          sh ' mvn clean install sonar:sonar '
          sh 'cat target/sonar/report-task.txt'
      }
    }
    
    stage ('Build') {
      steps {
      sh 'mvn compile war:war '
     // sh 'mvn clean package'
      }
    }
    
    stage ('Deploy-To-Tomcat') {
      steps {
                sh 'mvn install tomcat7:deploy'     
           }       
    }
    
    
    stage ('DAST') {
      steps {
        sshagent(['zap']) {
          sh 'mvn br.com.softplan.security.zap:zap-maven-plugin:analyze'
         sh 'ssh -P 8889 -o  StrictHostKeyChecking=no justdial@172.29.87.55 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://172.29.87.55:8899/my-application/" || true'
        }
      }
    }
    
  }
}
