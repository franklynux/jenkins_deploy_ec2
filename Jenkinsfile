pipeline {
    agent any

    environment {
        EC2_IP = '3.83.179.213'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials') // Ensure this ID matches your Jenkins credentials
        DOCKER_IMAGE = 'franklynux/e-commerce-web:${BUILD_NUMBER}'
        EC2_INSTANCE_KEY = credentials('ec2-server') // Ensure this ID matches your Jenkins SSH credentials
    }

    stages {
        stage('Fetch Code') {
            steps {
                script {
                    echo "Pulling source code from Git"
                    git branch: 'main', url: 'https://github.com/franklynux/jenkins_deploy_ec2.git'
                }
            }
        }
       
        stage('Unit Tests') {
            steps {
                echo "Running Unit Tests"
                sh 'python3 test_website.py'
            }
            post {
                always {
                    // Example: Publish test reports if generated
                    // Uncomment and modify the following lines if applicable
                    // junit 'test-results/*.xml'
                    
                    // Example: Publish HTML coverage reports
                    // publishHTML([
                    //     allowMissing: false,
                    //     alwaysLinkToLastBuild: true,
                    //     keepAll: true,
                    //     reportDir: 'coverage',
                    //     reportFiles: 'index.html',
                    //     reportName: 'Coverage Report',
                    //     reportTitles: 'Code Coverage'
                    // ])
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker Image: ${DOCKER_IMAGE}"
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "Pushing Docker Image to Docker Hub"
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                    // Optional: Remove the local image to save space
                    sh "docker rmi ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    echo "Deploying to EC2 Instance: ${EC2_IP}"
                    def dockerCmd = """
                        docker pull ${DOCKER_IMAGE}
                        docker stop e-commerce-web || true
                        docker rm e-commerce-web || true
                        docker run -d --name e-commerce-web -p 80:80 ${DOCKER_IMAGE}
                    """
                    sshagent(['ec2-server']) { // Ensure 'ec2-server' matches your SSH credentials ID
                        // Execute Docker commands on EC2
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} '${dockerCmd}'"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
            // Optionally, perform additional cleanup or logging
        }
        success {
            echo 'Pipeline completed successfully!'
            // Optionally, send a success notification (e.g., Slack, Email)
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
            // Optionally, send a failure notification
        }
    }
}
