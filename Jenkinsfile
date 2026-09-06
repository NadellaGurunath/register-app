pipeline{
	agent{ label 'jenkins-Agent'}
	tools{
		jdk 'Java17'
		maven 'Maven3'
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
		stage("SonarQube Analasis"){
			steps{
				script{
					withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token'){
						sh "mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar"
					}
				}
			}
		}
	}
}
