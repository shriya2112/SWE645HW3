pipeline {
  agent any
  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub') // Ensure 'dockerhub' is configured in Jenkins
  }
  stages {
    stage('Clone Repository') {
      steps {
        // Ensure the latest code is cloned
        checkout scm
      }
    }
    stage('Build Maven Project') {
      agent {
        // Use Maven Docker image for building
        docker {
          image 'maven:3.9.2'
        }
      }
      steps {
        // Build the Maven project, skipping tests
        sh 'mvn clean package -DskipTests'
        
        // Check the target directory for the built artifact
        sh 'ls -l target'
      }
    }
    stage('Build Docker Image') {
      steps {
        // Build the Docker image
        sh 'docker build -t satluri2/survey:latest .'
      }
    }
    stage('Login to Docker Hub') {
      steps {
        // Log in to Docker Hub using Jenkins credentials
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
        // Update Kubernetes deployment with the new image
        sh 'kubectl set image deployment/surveyassign3-deploy container-0=satluri2/survey:latest -n default'
        
        // Restart the Kubernetes deployment
        sh 'kubectl rollout restart deployment/surveyassign3-deploy -n default'
        
        // Verify rollout status
        sh 'kubectl rollout status deployment/surveyassign3-deploy -n default'
      }
    }
  }
  post {
    always {
      // Log out from Docker Hub after the pipeline
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
