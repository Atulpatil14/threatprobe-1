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
      
      stage ('Check secrets') {
          steps {
              sh 'trufflehog3 https://github.com/ajaym2812/threatprobe.git -f json -o truffelhog_output.json || true'
            //  sh './trufflehog_report.sh'
            }
          
        }
      
      stage ('Software composition analysis') {
          steps {
          dependencyCheck additionalArguments: ''' 
          -o "./" 
          -s "./"
          -f "ALL" 
          --prettyPrint''', odcInstallation: 'OWASP-DC'
          dependencyCheckPublisher pattern: 'dependency-check-report.xml'
              
          }
          
      }

      stage ('Static analysis') {
          steps {
              withSonarQubeEnv('sonarqube') {
                  sh 'mvn sonar:sonar'
				  }
				}
			}
			// stage ('Generate build') {
			//     steps {
			//         sh 'mvn clean install -DskipTests'
			        
			//     }
			// }
	  stage ('Deploy to server') {
            steps {
	   timeout(time: 3, unit: 'MINUTES') {
              sshagent(['app-server']) {
                sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/webgoat-devsecops/webgoat-server/target/webgoat-server-v8.2.0-SNAPSHOT.jar ubuntu@3.109.152.116:/WebGoat'
		sh 'ssh -o  StrictHostKeyChecking=no ubuntu@3.109.152.116 "nohup java -jar /WebGoat/webgoat-server-v8.2.0-SNAPSHOT.jar &"'
                  }
	     }
        }     
    }
}
}
