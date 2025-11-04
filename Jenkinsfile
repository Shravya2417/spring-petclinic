pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "docker.io"
        DOCKER_IMAGE = "shravya1207/petclinicpage1"
        SONAR_SERVER = "My-Sonar"
        SONAR_TOKEN_ID = "sonar-token"
        DOCKER_CREDS = "docker-hub-creds"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "📦 Cloning Repository..."
                git branch: 'main', url: 'https://github.com/Shravya2417/spring-petclinic.git'
            }
        }

        stage('Build & Test') {
            steps {
                echo "🔨 Building the Project..."
                sh "mvn clean package -DskipTests=false"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {
                    withCredentials([string(credentialsId: "${SONAR_TOKEN_ID}", variable: 'SONAR_TOKEN')]) {
                        echo "🔍 Running SonarQube code analysis..."
                        sh "mvn sonar:sonar -Dsonar.projectKey=spring-petclinic -Dsonar.host.url=https://35.154.173.51:9000 -Dsonar.login=${SONAR_TOKEN}"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "🚀 Pushing Docker image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                    docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Deploy to ArgoCD') {
            steps {
                echo "🚀 Triggering ArgoCD Deployment..."
                sh '''
                argocd app sync spring-petclinic --grpc-web
                argocd app wait spring-petclinic --timeout 300
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline executed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Please check logs."
        }
    }
}
