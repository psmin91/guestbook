pipeline {
    agent any
    
    stages {
        stage('Checkout') { steps { echo "Checkout"} }
        stage('Build') { steps { echo "Build"} }
        stage('Docker Image Build') { steps { echo "Docker Image Build"} }
        stage('Docker Image Push') { steps { echo "Docker Image Push"} }
        stage('Staging Deploy') { steps { echo "Staging Deploy"} }
        stage('PROD Deploy') {
            steps { 
                input "운영서버에 배포하시겠습니까?"
                } 
            }
    }
}
