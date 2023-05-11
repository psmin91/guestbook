import java.text.SimpleDateFormat

def TODAY = (new SimpleDateFormat("yyyyMMddHHmmss")).format(new Date())

pipeline {
    agent any
    environment {
        strDockerTag = "${TODAY}_${BUILD_ID}"
        strDockerImage ="psmin91/cicd_guestbook:${strDockerTag}"
    }
    stages {
        stage('Checkout') {
            steps {
                git credentialsId: '78a69571-9ce8-4e06-a9d5-2629710a9482', url: 'http://git.biz-think.net/system/jenkins_10059.git'
            }
        }
        stage('Build') {
            steps {
                sh './mvnw clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps{
                echo 'SonarQube Analysis'
                /*
                withSonarQubeEnv('SonarQube-Server'){
                    sh '''
                        ./mvnw sonar:sonar \
                        -Dsonar.projectKey=guestbook \
                        -Dsonar.host.url=http://192.168.56.143:9000 \
                        -Dsonar.login=21193ff67973f0efc068ac33ce547e3da8c671b7
                    '''
                }
                */
            }
        }
        stage('SonarQube Quality Gate'){
            steps{
                echo 'SonarQube Quality Gate'
                /*
                timeout(time: 1, unit: 'MINUTES') {
                    script{
                        def qg = waitForQualityGate()
                        if(qg.status != 'OK') {
                            echo "NOT OK Status: ${qg.status}"
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        } else{
                            echo "OK Status: ${qg.status}"
                        }
                    }
                }
                */
            }
        }
        stage('Docker Image Build') {
            steps {
                script {
                    //oDockImage = docker.build(strDockerImage)
                    oDockImage = docker.build(strDockerImage, "--build-arg VERSION=${strDockerTag} -f Dockerfile .")
                }
            }
        }
        stage('Docker Image Push') {
            steps {
                script {
                    docker.withRegistry('', 'DockerHub_psmin91') {
                        oDockImage.push()
                    }
                }
            }
        }
        stage('SSH Staging Server') {
            steps {
                sshagent(credentials: ['Staging-PrivateKey']) {
                    sh "ssh -o StrictHostKeyChecking=no root@43.201.72.54 docker container rm -f guestbookapp"
                    sh "ssh -o StrictHostKeyChecking=no root@43.201.72.54 docker container run \
                                        -d \
                                        -p 38080:80 \
                                        --name=guestbookapp \
                                        -e MYSQL_IP=3.35.53.130 \
                                        -e MYSQL_PORT=3306 \
                                        -e MYSQL_DATABASE=guestbook \
                                        -e MYSQL_USER=root \
                                        -e MYSQL_PASSWORD=education \
                                        ${strDockerImage} "
                }
            }
        }
        stage ('JMeter LoadTest') {
            steps { 
                sh '~/lab/sw/jmeter/bin/jmeter.sh -j jmeter.save.saveservice.output_format=xml -n -t src/main/jmx/guestbook_loadtest.jmx -l loadtest_result.jtl' 
                perfReport filterRegex: '', showTrendGraphs: true, sourceDataFiles: 'loadtest_result.jtl' 
            } 
        }
    }
    post { 
        always { 
            slackSend(tokenCredentialId: 'slack-token'
                , channel: '#edu'
                , color: 'good'
                , message: "${JOB_NAME} (${BUILD_NUMBER}) 빌드가 끝났습니다. Details: (<${BUILD_URL} | here >)")
        }
        success { 
            slackSend(tokenCredentialId: 'slack-token'
                , channel: '#edu'
                , color: 'good'
                , message: "${JOB_NAME} (${BUILD_NUMBER}) 빌드가 성공적으로 끝났습니다. Details: (<${BUILD_URL} | here >)")
        }
        failure { 
            slackSend(tokenCredentialId: 'slack-token'
                , channel: '#edu'
                , color: 'danger'
                , message: "${JOB_NAME} (${BUILD_NUMBER}) 빌드가 실패하였습니다. Details: (<${BUILD_URL} | here >)")
        }
    }
}
