pipeline {
    agent any
    tools {
        maven 'Maven 3.9.2' // Use the configured Maven tool
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') // Docker credentials
    }
    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }
        stage('Build Maven Project') {
            steps {
                // Build the Maven project
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t satluri2/survey:latest .'
            }
        }
        stage('Login to Docker Hub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Push Docker Image to Docker Hub') {
            steps {
                sh 'docker push satluri2/survey:latest'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl set image deployment/surveyassign3-deploy container-0=satluri2/survey:latest -n default'
                sh 'kubectl rollout restart deployment/surveyassign3-deploy -n default'
                sh 'kubectl rollout status deployment/surveyassign3-deploy -n default'
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
