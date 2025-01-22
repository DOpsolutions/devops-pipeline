pipeline {
    agent any

    environment {
        // Environment variables for Docker registry
        DOCKER_IMAGE = "myapp"
        REGISTRY_URL = "docker.io"
        REPO_NAME = "username/myapp"
        SLACK_CHANNEL = "#devops-notifications"
        SLACK_TOKEN = credentials('slack-token')  // Jenkins Slack plugin credential
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the latest code from the repository
                    git branch: 'main', url: 'https://github.com/username/myapp.git'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    echo "Building Docker image"
                    sh """
                    docker build -t $REGISTRY_URL/$REPO_NAME:$BUILD_NUMBER .
                    """
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                script {
                    // Run unit tests
                    echo "Running unit tests"
                    sh "docker run --rm $REGISTRY_URL/$REPO_NAME:$BUILD_NUMBER npm run test"
                }
            }
        }

        stage('Static Code Analysis') {
            steps {
                script {
                    // Run static code analysis with SonarQube (example)
                    echo "Running static analysis"
                    sh """
                    sonar-scanner -Dsonar.projectKey=myapp -Dsonar.sources=. -Dsonar.host.url=http://sonarqube:9000
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push Docker image to the registry
                    echo "Pushing Docker image to registry"
                    sh """
                    docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                    docker push $REGISTRY_URL/$REPO_NAME:$BUILD_NUMBER
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy the app to a Kubernetes cluster
                    echo "Deploying to Kubernetes"
                    sh """
                    kubectl set image deployment/myapp-deployment myapp=$REGISTRY_URL/$REPO_NAME:$BUILD_NUMBER
                    kubectl rollout status deployment/myapp-deployment
                    """
                }
            }
        }

        stage('Monitor') {
            steps {
                script {
                    // Simple monitoring: Check app health (e.g., using Kubernetes)
                    echo "Monitoring app health"
                    sh "kubectl get pods -l app=myapp"
                }
            }
        }

        stage('Notify') {
            steps {
                script {
                    // Send a Slack notification on pipeline status (success/failure)
                    if (currentBuild.result == 'SUCCESS') {
                        slackSend(channel: SLACK_CHANNEL, message: "Build #${BUILD_NUMBER} succeeded and deployed!")
                    } else {
                        slackSend(channel: SLACK_CHANNEL, message: "Build #${BUILD_NUMBER} failed!")
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up after the pipeline finishes (e.g., remove local Docker images)
            sh "docker system prune -f"
        }
        failure {
            // Notify failure via Slack
            slackSend(channel: SLACK_CHANNEL, message: "Build #${BUILD_NUMBER} failed!")
        }
        success {
            // Notify success via Slack
            slackSend(channel: SLACK_CHANNEL, message: "Build #${BUILD_NUMBER} succeeded!")
        }
    }
}

