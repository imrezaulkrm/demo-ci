pipeline {
    agent any
    environment {
        DOCKERHUB_USERNAME = "imrezaulkrm"
        APP_NAME = "demo-ci-cd"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
        REGISTRY_CREDS = 'dockerhub'
        }
    stages {
        stage('Cleanup Workspace'){
            steps {
                script {
                    cleanWs()
                }
            }
        }
        stage('source code pull from github') {
            steps {
                git branch: 'main', url: 'https://github.com/imrezaulkrm/demo-ci.git'
            }
        }
        stage('Build Docker Image'){
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }
        stage('Push Docker Image'){
            steps {
                withCredentials([usernamePassword(credentialsId: 'Dockerhub', passwordVariable: 'dpass', usernameVariable: 'dockeruser')]) {
                    sh "docker login -u $dockeruser --password $dpass"
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Delete Docker Images'){
            steps {
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker rmi ${IMAGE_NAME}:latest"
                sh "cd .."
            }
        }
        stage('k8s source code pull from github') {
            steps {
                git branch: 'main', url: 'https://github.com/imrezaulkrm/demo-cd.git'
            }
        }
        stage('Updating Kubernetes deployment file') {
            steps {
                sh "cd demo-cd"
                sh "cat k8s/demo-deployment.yaml"
                // Construct the sed command to change only line 18
                sh """sed -i '18s#image:.*#image: ${IMAGE_NAME}:${IMAGE_TAG}#' k8s/demo-deployment.yaml"""
                sh "cat k8s/demo-deployment.yaml"
            }
        }

        
        stage('Push the changed deployment file to Git') {
            steps {
                script {
                    sh 'git config --global user.name "imrezaulkrm"'
                    sh 'git config --global user.email "sayem010ahmed@gmail.com"'
                    sh 'git add .'
                    sh 'git commit -m "Updated the deployment file"'
                    withCredentials([usernamePassword(credentialsId: 'Github', passwordVariable: 'gpass', usernameVariable: 'githubuser')]) {
                        sh "git push https://$githubuser:$gpass@github.com/imrezaulkrm/demo-cd.git main"
                    }
                }
            }
        }
    }
}