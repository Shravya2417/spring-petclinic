pipeline {
  agent any

  environment {
    DOCKER_REGISTRY = "docker.io"                                // Docker Hub registry
    DOCKER_REPO     = "shravya1207/petclinicpage1"          // 🔁 REPLACE with your Docker Hub repo name
    SONAR_SERVER    = "My-Sonar"                                 // 🔁 REPLACE with SonarQube server name (Manage Jenkins -> System)
    SONAR_TOKEN_ID  = "sonar-token"                              // 🔁 REPLACE with Jenkins credentials ID for SonarQube token
    DOCKER_CRED_ID  = "docker-hub-creds"                         // 🔁 REPLACE with Jenkins credentials ID for Docker Hub
  }

  tools {
    jdk 'jdk25'          // make sure JDK 17 is configured in Manage Jenkins → Global Tool Configuration
    maven 'maven3'       // same for Maven
  }

  stages {

    stage('Checkout') {
      steps {
        echo "📦 Checking out code from GitHub..."
        checkout scm
      }
    }

    stage('Build') {
      steps {
        echo "🔨 Building project with Maven..."
        sh 'mvn -B -DskipTests clean package'
        archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
      }
    }

    stage('SonarQube Analysis') {
      steps {
        echo "🔍 Running SonarQube code analysis..."
        withSonarQubeEnv("${SONAR_SERVER}") {
          sh "mvn -B sonar:sonar -Dsonar.host.url=http://52.66.197.131:9000 -Dsonar.login=$SONAR_TOKEN"
          // 🔁 REPLACE <EC2_PUBLIC_IP> with your SonarQube server IP
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          def tag = "${env.BUILD_NUMBER}"
          echo "🐳 Building Docker image: ${DOCKER_REGISTRY}/${DOCKER_REPO}:${tag}"
          sh "docker build -t ${DOCKER_REGISTRY}/${DOCKER_REPO}:${tag} ."
          sh "docker tag ${DOCKER_REGISTRY}/${DOCKER_REPO}:${tag} ${DOCKER_REGISTRY}/${DOCKER_REPO}:latest"
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        echo "📤 Pushing Docker image to Docker Hub..."
        withCredentials([usernamePassword(credentialsId: "${DOCKER_CRED_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin ${DOCKER_REGISTRY}'
          sh "docker push ${DOCKER_REGISTRY}/${DOCKER_REPO}:${env.BUILD_NUMBER}"
          sh "docker push ${DOCKER_REGISTRY}/${DOCKER_REPO}:latest"
        }
      }
    }

    stage('Update Manifests (for ArgoCD)') {
      steps {
        echo "📝 (Optional) Update manifests repo for ArgoCD to auto-sync new image."
        echo "You can automate Git commit/push of updated deployment.yaml here."
      }
    }
  }

  post {
    always {
      echo "📊 Publishing test results..."
      junit 'target/surefire-reports/*.xml'
    }
  }
}
