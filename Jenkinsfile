pipeline {
  agent any
  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub') // Ensure 'dockerhub' is configured in Jenkins
  }
  stages {
    stage('Clone Repository') {
      steps {
        // Clone the repository to ensure we have the latest code
        checkout scm
      }
    }
    stage('Build Maven Project') {
      steps {
        // Clean and build the Maven project, skipping tests
        sh 'mvn clean package -DskipTests'
        
        // Check the target directory for the built artifact
        sh 'ls -l target'
      }
    }
    stage('Build Docker Image') {
      steps {
        // Build the Docker image using the Dockerfile
        sh 'docker build -t satluri2/survey:latest .'
      }
    }
    stage('Login to Docker Hub') {
      steps {
        // Log in to Docker Hub using credentials stored in Jenkins
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
      }
    }
    stage('Push Docker Image to Docker Hub') {
      steps {
        // Push the Docker image to Docker Hub
        sh 'docker push satluri2/survey:latest'
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        // Update the Kubernetes deployment with the new Docker image
        sh 'kubectl set image deployment/surveyassign3-deploy container-0=satluri2/survey:latest -n default'
        
        // Roll out the updated deployment
        sh 'kubectl rollout restart deployment/surveyassign3-deploy -n default'
        
        // Ensure the deployment is successful
        sh 'kubectl rollout status deployment/surveyassign3-deploy -n default'
      }
    }
  }
  post {
    always {
      // Log out from Docker Hub
      sh 'docker logout'
    }
    success {
      echo 'Pipeline completed successfully!'
    }
    failure {
      echo 'Pipeline failed. Check the logs for details.'
    }
  }
}
