pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'osamaanas'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        EC2_UAT_IP = '54.221.29.199'
        EC2_PROD_IP = '13.218.127.226'
        SSH_CREDENTIALS_ID = 'aws-ssh-key'
        DOCKER_IMAGE = "${osamaanas}/dotnet-hello-world"
    }

    parameters {
        choice(name: 'DEPLOY_TARGET', choices: ['UAT', 'PROD'], description: 'Select the deployment target environment.')
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${env.BUILD_NUMBER} ."
                    echo "Docker image built: ${DOCKER_IMAGE}:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", passwordVariable: 'Anas@9834', usernameVariable: 'osamaanas')]) {
                        sh "echo ${Anas@9834} | docker login -u ${osamaanas} --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}:${env.BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    def deployIp = (params.DEPLOY_TARGET == 'UAT') ? env.EC2_UAT_IP : env.EC2_PROD_IP

                    sshagent(credentials: ["${SSH_CREDENTIALS_ID}"]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${deployIp} "
                            sudo docker stop dotnet-app || true
                            sudo docker rm dotnet-app || true
                            sudo docker pull ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                            sudo docker run -d --restart always -p 80:80 --name dotnet-app ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                        "
                        """
                    }
                }
            }
        }
    }
}
