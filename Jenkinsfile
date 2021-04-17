pipeline {
    agent any
    stages {
        stage('Build Backend') {
           steps {
               bat 'mvn clean package -DskipTests=True'
           }
        }
        stage('Unit Tests') {
           steps {
               bat 'mvn test'
           }
        }
        stage('Sonar Analysis') {
        	environment {
	            scannerHome = tool 'SONAR_SCANNER'
	        }        	        	
           	steps {
           		withSonarQubeEnv('SONAR_LOCAL'){
    				bat "\"${scannerHome}/bin/sonar-scanner\" -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=9fb63982ce7a44d1953b9bbfbf307fa86b23b1ff -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"       		    
           		}
            }
        }
        stage('Quality Gate') {        	       	
           	steps {
           		sleep(50)
				timeout(time: 1, unit: 'MINUTES'){
				    waitForQualityGate abortPipeline: true                 
				}
            }
        }
    }
}