// when과 parameter를 이용하여 특정 Stage 건너뛰기
pipeline {
    agent any
    parameters {
        booleanParam(name: 'BUILD_DOCKER_IMAGE', 
                    defaultValue: true, 
                    description: 'Docker Image Build')
    }

    stages {
        stage('Docker Image Build') {
            when { expression { return params.BUILD_DOCKER_IMAGE }} 
            steps {
                echo 'Docker Image Build...'
            }
        }
    }
}

