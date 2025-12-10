pipeline {
    agent any

    tools {
        nodejs 'node16'
        jdk 'jdk17' // Optional, if needed for other tools
    }

    environment {
        SCANNER_HOME = tool 'mysonar'
        AWS_REGION = 'ap-south-1'
        ECR_REPO_APP = '904923506382.dkr.ecr.us-east-1.amazonaws.com/ravi031/myzomato'
    }

    stages {
        stage("Clean") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout Code") {
            steps {
                git branch: 'main', url: 'https://github.com/RaviVarma06/Zomato-Repo.git'
            }
        }

        stage("Install Dependencies") {
            steps {
                sh 'npm install'
            }
        }

        stage("Code Quality Analysis") {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=docker-webapp \
                        -Dsonar.projectName=docker-webapp \
                        -Dsonar.sources=. \
                        -Dsonar.login=sqa_c57711f61c5845601a8678702ba858b460cb3b1b

                    '''
                }
            }
        }

        stage("Docker Build & Tag") {
            steps {
                script {
                    sh "docker build -t ravi031/myzomato ."
                    sh "docker tag ravi031/myzomato $ECR_REPO_APP"
                }
            }
        }

        stage("Push to ECR") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'AWS-ECR-CREDS', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set default.region $AWS_REGION
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO_APP
                        docker push $ECR_REPO_APP
                    '''
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                sh 'trivy fs . > trivyfs.txt'
                sh "trivy image $ECR_REPO_APP"
            }
        }

        stage("Deploy to Container") {
            steps {
                sh "docker run -d --name zomato -p 3000:3000 $ECR_REPO_APP"
            }
        }
    }
}
