pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'mysonar'
    }

    stages {
        stage("Clean") {
            steps {
                cleanWs()
            }
        }

        stage("Code") {
            steps {
                git branch: 'main', url: 'https://github.com/RaviVarma06/Zomato-Repo.git'
            }
        }

        stage("CQA") {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=zomato \
                        -Dsonar.projectKey=zomato'''
                }
            }
        }

        stage("Install dependencies") {
            steps {
                sh 'npm install'
            }
        }

        stage("Image Build") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'DOCKER-HUB') {
                        sh "docker build -t ravi031/myzomato:v1 ."
                    }
                }
            }
        }

        stage("TrivyScan") {
            steps {
                sh 'trivy fs . > trivyfs.txt'
                sh 'trivy image ravi031/myzomato:v1'
            }
        }

        stage("DHpush") {
            steps {
                withDockerRegistry(credentialsId: 'DOCKER-HUB', url: 'https://index.docker.io/v1/') {
                    sh 'docker push ravi031/myzomato:v1'
                }
            }
        }

        stage("Deploy to container"){
            steps{
	             sh 'docker run -d --name zomato -p 3000:3000 ravi031/myzomato:v1'
	          }
      	}
    }
     post {
        always {
            echo 'Pipeline execution completed.'
            cleanWs()
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
