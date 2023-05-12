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
                git branch: 'main', url:'https://github.com/psmin91/guestbook.git'

                dir('/home/ec2-user/lab/projects/guestbook-config'){
                    git branch: 'master', url:'https://github.com/yu3papa/guestbook-config.git'
                }
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package'
            }
        }

        stage('Docker Image Build') {
            steps {
                script {
                    oDockImage = docker.build(strDockerImage, "--build-arg VERSION=${strDockerTag} -f Dockerfile .")
                }
            }
        }

        stage('Docker Image Push') {
            steps {
                script {
                    docker.withRegistry('', 'docker_hub') {
                        oDockImage.push()
                    }
                }
            }
        }

        stage('Config-Repo PUSH') {
            environment {
                GITHUB_ACCESS_TOKEN = credentials('github_token')
            }
            steps {
                dir('/home/ec2-user/lab/projects/guestbook-config'){
                    sh '''
                        sed -i "s/cicd_guestbook:.*/cicd_guestbook:${strDockerTag}/g" guestbook/guestbook_deploy.yaml
                        git add guestbook/guestbook_deploy.yaml
                        git commit -m "[UPDATE] guestbook image tag - ${strDockerImage} (by jenkins)"
                        git push "https://psmin91:${GITHUB_ACCESS_TOKEN}@github.com/psmin91/guestbook-config.git"
                    '''
                }
            }
        }

        stage('ArgoCD Sync') {
            environment {
                ARGOCD_API_TOKEN = credentials('argocd-api-token')
            }
            steps {
                sh '''
                    TOKEN="${ARGOCD_API_TOKEN}"
                    PAYLOAD='{"prune": true}'
                    curl -v -k -XPOST \
                        -H "Authorization: Bearer ${TOKEN}" \
                        https://43.202.33.140/api/v1/applications/guestbook/sync
                '''
            }
        }
    }
}
