pipeline {
  agent any

  environment {
    DOCKER_REGISTRY = "docker.io"                      // or your registry
    DOCKER_REPO     = "shravya1207/petclinicpage1"    // REPLACE: your dockerhub repo
    SONAR_SERVER    = "My-Sonar"                       // Jenkins Sonar name
    SONAR_TOKEN_ID  = "squ_f33589f1aac7bb7e2364903d06796a66f86d64da"                    // Jenkins credentials ID for Sonar token
    DOCKER_CRED_ID  = "docker-hub-creds"               // Jenkins credentials id for Docker Hub (username/password)
  }

  tools {
    jdk 'jdk17'    // configure this tool name in Manage Jenkins -> Global Tool Configuration
    maven 'maven3' // same: configure Maven in Jenkins tools
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build') {
      steps {
        sh 'mvn -B -DskipTests clean package'
        archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv("${SONAR_SERVER}") {
          // call sonar-scanner installed as tool or use mvn sonar:sonar
          sh "mvn -B sonar:sonar -Dsonar.host.url=http://3.110.62.234:9000 -Dsonar.login=\$SONAR_TOKEN"
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          def tag = "${env.BUILD_NUMBER}"
          sh "docker build -t ${DOCKER_REGISTRY}/${DOCKER_REPO}:${tag} ."
          sh "docker tag ${DOCKER_REGISTRY}/${DOCKER_REPO}:${tag} ${DOCKER_REGISTRY}/${DOCKER_REPO}:latest"
        }
      }
    }

    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_CRED_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin ${DOCKER_REGISTRY}'
          sh "docker push ${DOCKER_REGISTRY}/${DOCKER_REPO}:${env.BUILD_NUMBER}"
          sh "docker push ${DOCKER_REGISTRY}/${DOCKER_REPO}:latest"
        }
      }
    }

    stage('Update Manifests (optional)') {
      steps {
        // Option: update a manifests repo or kustomize image tag file so ArgoCD picks new image
        echo "Add script here to update your manifests repo (git clone/edit/push)"
      }
    }
  }

  post {
    always {
      junit 'target/surefire-reports/*.xml'
    }
  }
}

