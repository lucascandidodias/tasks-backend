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
    				bat "\"${scannerHome}/bin/sonar-scanner\" -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=9fb63982ce7a44d1953b9bbfbf307fa86b23b1ff -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java,**/bin/frontend/**"       		    
           		}
            }
        }
        stage('Quality Gate') {        	       	
           	steps {
           		sleep(5)
				timeout(time: 1, unit: 'MINUTES'){
				    waitForQualityGate abortPipeline: true                 
				}
            }
        }
        stage('Deploy Backend') {        	       	
           	steps {
				deploy adapters: [tomcat8(credentialsId: 'TomacatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage('API Test') {        	       	
           	steps {
           		dir('api-test'){
					git credentialsId: 'github_login', url: 'https://github.com/lucascandidodias/tasks-api-test.git'
	 				bat 'mvn test'           		    
           		}
            }
        }
         stage('Deploy Frontend') {        	       	
           	steps {
         		dir('frontend'){
	         		git credentialsId: 'github_login', url: 'https://github.com/lucascandidodias/tasks-frontend.git'
	         		bat 'mvn clean package'
					deploy adapters: [tomcat8(credentialsId: 'TomacatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
				}
            }
        }
        stage('Funcional Test') {        	       	
           	steps {
           		dir('funtional-test'){
					git credentialsId: 'github_login', url: 'https://github.com/lucascandidodias/tasks-funcional-tests.git'
	 				bat 'mvn test'           		    
           		}
            }
        }        
         stage('Deploy Prod') {        	       	
           	steps {
           		bat 'docker-compose build'
           		bat 'docker-compose up -d'        		               		
            }
        }
        stage('Health Check') {        	       	
           	steps {
           		sleep(5)
           		dir('funtional-test'){					
	 				bat 'mvn verify -Dskip.surefire.tests'           		    
           		}
            }
        }        
    }
     post { 
        always { 
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, funtional-test/target/surefire-reports/*.xml, funtional-test/target/failsafe-reports/*.xml'
        }
    }

}


