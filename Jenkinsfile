pipeline {
    agent any 
    options {
        disableConcurrentBuilds()
    }

    environment {
        DOCKER_IMAGE = "srinu0930/multibranch-flask-app"
        GIT_USERNAME = "srinucloud"
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
                env.imageTag = "build-${BUILD_NUMBER}"
                withDockerRegistry(credentialsId: 'docker-creds',
                usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD', 
                toolName: 'docker', url: 'https://index.docker.io/v1/') {
                    sh "docker build -t ${DOCKER_IMAGE}:${imageTag} ."
                    echo "$DOCKER_PASSWORD" | docker login -u $DOCKER_USERNAME --password-stdin
                    sh "docker push ${DOCKER_IMAGE}:${imageTag}"
                }

            }
        }

        stage('Deploy to Kubernetes') {
            when {
                branch 'main'
            }

            steps {
                script {
                    withCredentials([gitUsernamePassword(credentialsId: 'github-creds',usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT-TOKEN',   gitToolName: 'Default')]) {
                        
                        sh ''' 
                        set -e
                        git config --global user.name '${GIT_USERNAME}'
                        git config --global user.email '${GIT_EMAIL}'

                        git fetch origin main
                        git checkout main
                        git reset --hard origin/main

                        sed -i "s|image: .*|image: ${DOCKER_IMAGE}:${imageTag}|g" k8s/deployment.yml
                        git add k8s/deployment.yml
                        git diff --cached --quiet || git commit -m "Update deployment image to ${DOCKER_IMAGE}:${imageTag}"
                        git push origin main
                        '''

                }
            }
        }
    }

    }
}