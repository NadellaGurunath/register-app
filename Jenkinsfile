pipeline{
	agent{ label 'jenkins-Agent'}
	tools{
		jdk 'Java17'
		maven 'Maven3'
	}
	environment{
		APP_NAME = "register-app-pipeline
		RELEASE = "1.0.0"
		DOCKER_USER = gurunath486
		DOCKER_PASS = 'dockerhub'
		IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
		IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	}
	stages{
		stage("Cleanup Workspace"){
			steps{
				cleanWs()
			}
		}
		stage("Checkout from SCM"){
			steps{
				git branch: 'main', credentialsId: 'github', url: 'https://github.com/NadellaGurunath/register-app'
			}
		}
		stage("Build Application"){
			steps{
				sh "mvn clean package"
			}
		}
		stage("Test Application"){
			steps{
				sh "mvn test"
			}
		}
		stage("SonarQube Analysis") {
		    steps {
		        withSonarQubeEnv(
		            installationName: 'sonarqube-server',
		            credentialsId: 'jenkins-sonarqube-token'
		        ) {
		            sh '''
		                export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
		                export PATH=$JAVA_HOME/bin:$PATH
		
		                java -version
		
		                mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.1.2184:sonar
		            '''
		        }
    		}
		}

		stage("Quality gate"){
			steps{
				script{
					waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
				}	
			}
		}
		stage("Build and push docker image"){
			steps{
				script{
					docker.withRegistry('',DOCKER_PASS){
						docker_image.push("${IMAGE_TAG}")
						docker_image.push(latest)
					}
				}
			}
		}
	}
}
