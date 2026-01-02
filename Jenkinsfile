pipeline {
    agent any
    
    options {
        disableConcurrentBuilds()
    }
    
    environment {
        IMAGE_NAME = "sekhar295/multibranch-flask-app"
        GIT_USER = "sekhar295"
        GIT_EMAIL = "mscsekhar123@gmail.com"
    }
    
    stages{
        stage ("Git Checkout"){
            steps {
                checkout scm
            }
        }
        stage ("Build and push Image"){
            when { branch 'main' }
            steps {
                script {
                    
                    env.IMAGE_TAG = "build-${BUILD_NUMBER}"
                    
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerpass',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }
        stage ("Update K8S Manifest"){
            when { branch 'main' }
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'githubpass',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {
                        sh """
                        set -e
                        git config user.name "$GIT_USER"
                        git config user.email "$GIT_EMAIL"
                        git fetch origin 
                        git checkout main
                        git reset --hard origin/main
                        sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yml
                        git add k8s/deployment.yml
                        git diff --cached --quiet || git commit -m "Updated Image to ${IMAGE_TAG}"
                        git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/Sekhar295/production-grade-deployment.git main
                        """
                    }
                }
            }
        }
    }
}
