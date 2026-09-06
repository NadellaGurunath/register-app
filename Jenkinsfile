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
		stage("SonarQube Analysis") {
		    steps {
		        withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
		            sh '''
		                export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
		                export PATH=$JAVA_HOME/bin:$PATH
		
		                java -version
		
		                mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.1.2184:sonar
		            '''
		        }
	    	}
		}
	}
}
