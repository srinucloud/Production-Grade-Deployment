pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        DOCKER_IMAGE = "srinu0930/multibranch-flask-app"
        GIT_EMAIL = "srinuengr@gmail.com"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Docker Image') {
            when {
                branch 'main'
            }

            steps {
                script {
                    env.IMAGE_TAG = "build-${BUILD_NUMBER}"
                }

                withDockerRegistry(credentialsId: 'docker-creds', url: 'https://index.docker.io/v1/') {
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                branch 'main'
            }

            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'github-creds',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {

                        sh """
                            set -e

                            git config --global user.name "${GIT_USERNAME}"
                            git config --global user.email "${GIT_EMAIL}"

                            git fetch origin main
                            git checkout main
                            git reset --hard origin/main

                            sed -i 's|image: .*|image: ${DOCKER_IMAGE}:${IMAGE_TAG}|' k8s/deployment.yml

                            git add k8s/deployment.yml

                            git diff --cached --quiet || git commit -m "Update image to ${DOCKER_IMAGE}:${IMAGE_TAG}"

                            git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/srinucloud/Production-Grade-Deployment.git main
                        """
                    }
                }
            }
        }
    }
}
