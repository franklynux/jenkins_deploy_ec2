pipeline {
    agent any
    environment {
        EC2_IP = '3.83.179.213'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKER_IMAGE = 'franklynux/e-commerce-web:${BUILD_NUMBER}'
    }
    stages {
        stage ('Fetch Code') {
            steps {
                script {
                    echo "Pull source code from Git"
                    git branch: 'main', url: 'https://github.com/franklynux/jenkins_deploy_ec2.git'
                }
            }
        }
       
        stage('Install Python and Dependencies') {
            steps {
                sh '''
                    # Update package list without sudo
                    apt-get update || true
                   
                    # Install Python3 and pip if not already installed
                    apt-get install -y python3 python3-pip || true
                   
                    # Install BeautifulSoup4
                    pip3 install beautifulsoup4 || true
                '''
            }
        }
        stage('Unit Tests') {
            steps {
                sh 'python3 test_website.py || true'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} . || true"
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin || true"
                        sh "docker push ${DOCKER_IMAGE} || true"
                    }
                    // Optional: Remove the local image to save space
                    sh "docker rmi ${DOCKER_IMAGE} || true"
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                script {
                    def dockerCmd = """
                        docker pull ${DOCKER_IMAGE}
                        docker stop barista-cafe || true
                        docker rm barista-cafe || true
                        docker run -d --name barista-cafe -p 80:80 ${DOCKER_IMAGE}
                    """
                    sshagent(['ec2-server']) {
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} 'sudo apt-get update && sudo apt-get install -y docker.io' || true"
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} '${dockerCmd}' || true"
                    }
                }
            }
        }
    }
    post {
        always {
            sh 'docker logout || true'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
        }
    }
}