pipeline {
    agent any
    environment{
        APPROVAL_NO = ''
    }

    stages {
        stage('Checkout') { steps { echo "Checkout"} }
        stage('Build') { steps { echo "Build"} }
        stage('Docker Image Build') { steps { echo "Docker Image Build"} }
        stage('Docker Image Push') { steps { echo "Docker Image Push"} }
        stage('Staging Deploy') { steps { echo "Staging Deploy"} }
        stage('PROD Deploy') {

            steps { 
                script {
                    APPROVAL_NO = input message: '운영서버에 배포하려면 승인번호를 입력하세요',
                                        parameters: [string('승인번호')]

                    echo "${APPROVAL_NO}"
                }
            }
        }
    }
}
