pipeline {
    agent any
    environment {
        IMAGE_TAG = "v${BUILD_NUMBER}"
        SONAR_HOST_URL = 'http://172.30.85.64:9000/' 
        SONAR_SCANNER = tool 'sonarscanner'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/CharanReddy129/tictactoe.git'
            }
        }
        stage('sonar scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar', variable: 'SONAR_LOGIN')]) {
                    withSonarQubeEnv('sonarqube') {
                        sh """
                        ${SONAR_SCANNER}/bin/sonar-scanner \
                            -Dsonar.projectKey=tictactoe \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_LOGIN}
                        """
                    }
                }
            }
        }
        stage('docker build'){
            environment {
                IMAGE_TAG = "v${BUILD_NUMBER}"
            }
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh """
                    docker build -t charanreddy12/tictactoe:${IMAGE_TAG} .
                    docker push charanreddy12/tictactoe:${IMAGE_TAG}
                    """
                }
            }
        }
        stage('trivy image scan') {
            steps {
                sh "trivy image charanreddy12/tictactoe:${IMAGE_TAG}"
            }
        }
        stage('change image tag') {
            environment {
                GITHUB_CREDENTIALS = credentials('github_pat')
                GITHUB_USER_NAME = "CharanReddy129"
                GITHUB_REPO_NAME = 'tictactoe'
            }
            steps {
                sh """
                    git config --global user.name "charan"
                    git config --global user.mail "gajulapallicharan@gmail.com"
                    sed -i "s|charanreddy12/tictactoe:.*|charanreddy12/tictactoe:${IMAGE_TAG}|g" ./kubernetes/deployment.yml
                    git commit -am "changing the image tag to ${IMAGE_TAG}"
                    git push https://${GITHUB_CREDENTIALS}@github.com/${GITHUB_USER_NAME}/${GITHUB_REPO_NAME} HEAD:main
                """
            }
        }
    }
}
