pipeline{
	agent{ label 'jenkins-Agent'}
	tools{
		jdk 'Java17'
		maven 'Maven3'
	}
	environment{
		APP_NAME = "register-app-pipeline"
		RELEASE = "1.0.0"
		DOCKER_USER = 'gurunath486'
		DOCKER_PASS = 'dockerhub'
		IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
		IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
		JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
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
					def docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
					
					docker.withRegistry('',DOCKER_PASS){
						docker_image.push("${IMAGE_TAG}")
						docker_image.push("latest")
					}
				}
			}
		}
		stage("Trivy Scan") {
           steps {
               script {
	            	sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image gurunath486/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       	}
		stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }

		stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user gurunath:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-54-237-183-227.compute-1.amazonaws.com/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'"
                }
            }
       }
	}
}
