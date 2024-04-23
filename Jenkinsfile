pipeline {
    agent any
    environment {
        DOCKERHUB_USERNAME = "yasser744"
        APP_NAME = "devops-demo-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
        REGISTRY_CREDS = 'yasser_dockerhub'
        }
    stages {
        stage('Cleanup Workspace'){
            steps {
                script {
                    cleanWs()
                }
            }
        }
        stage('Checkout SCM on github'){
            steps {
                git credentialsId: 'github', 
                url: 'https://github.com/YasserAhmedMoh/gitops-demo.git',
                branch: 'test'
            }
        }
        stage('Build Docker Image'){
            steps {
                script{
                    docker_image = docker.build "${IMAGE_NAME}"
                }
            }
        }
        //    PUSH DOCKER IMAGE TO DOCKERHUB
        stage('Push Docker Image To DockerHub'){
            steps {
                script{
                    docker.withRegistry('', REGISTRY_CREDS ){
                        docker_image.push("${BUILD_NUMBER}")
                        docker_image.push('latest')
                    }
                }
            }
        } 
        stage('Push Docker Image To JCR') {
            steps {
                sh "docker login -u cadmin -p P@ssw0rd http://192.168.215.187:8082/artifactory/docker_jfrog_repo/"
                sh "docker build -f Dockerfile . -t  192.168.215.187:8082/docker_jfrog_repo/${IMAGE_NAME}:${BUILD_NUMBER}"
                sh "docker push 192.168.215.187:8082/docker_jfrog_repo/${IMAGE_NAME}:${BUILD_NUMBER}"
            }
        }
        stage('Delete Docker Images'){
            steps {
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker rmi ${IMAGE_NAME}:latest"
            }
        }
        stage('Updating Kubernetes deployment file'){
            steps {
                sh "cat deployment.yml"
                sh "sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yml"
                sh "cat deployment.yml"
            }
        }
        stage('Push the changed deployment file to Git'){
            steps {
                script{
                    sh """
                    git config --global user.name "YasserAhmedMoh"
                    git config --global user.email "ya81301@gmail.com"
                    git add deployment.yml
                    git commit -m 'Updated the deployment file' """
                    withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'pass', usernameVariable: 'user')]) {
                        sh "git push https://${user}:${pass}@github.com/YasserAhmedMoh/gitops-demo.git main"
                    }
                }
            }
        }
    }
}


