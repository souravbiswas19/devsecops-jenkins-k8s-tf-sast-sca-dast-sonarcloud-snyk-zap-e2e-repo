pipeline {
  agent any
  tools { 
        maven 'Maven_3_8_4'  
    }
	environment {
        SONAR_PROJECT_KEY = 'souravbuggywebapp'
        SONAR_ORGANIZATION = 'souravbuggywebapp'
        SONAR_HOST_URL = 'https://sonarcloud.io'
    }
   stages{
    stage('SonarCloud Analysis') {
            steps {
                echo 'Starting SonarCloud analysis...'
        
                withCredentials([
                    string(
                        credentialsId: 'sonarcloud-token',
                        variable: 'SONAR_TOKEN'
                    )
                ]) {
                    sh '''
                        mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.8.0.7211:sonar \
                        -Dsonar.projectKey=souravbuggywebapp \
                        -Dsonar.organization=souravbuggywebapp \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.token="$SONAR_TOKEN" \
                        -Dsonar.nodejs.executable=/usr/bin/node-22
                    '''
                }
            }
        }

	stage('RunSCAAnalysisUsingSnyk') {
            steps {		
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }

	stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("testing-ecr")
                 }
               }
            }
    }

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://335523638368.dkr.ecr.ap-southeast-2.amazonaws.com', 'ecr:ap-southeast-2:aws-credentials') {
                    app.push("latest")
                    }
                }
            }
    	}
	stage('Kubernetes Deployment of Sourav Bugg Web Application') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n devsecops')
		  sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
		}
	      }
   	} 
  }
}
