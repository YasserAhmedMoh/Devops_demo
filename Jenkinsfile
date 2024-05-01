def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
pipeline {
   // agent any
    agent {label 'jenkins_jcr_env'}
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
                url: 'https://github.com/YasserAhmedMoh/Devops_demo.git',
                branch: 'test'
            }
        }
        stage('Build Docker Image'){
            steps {
                sh 'docker build -f Dockerfile -t ${IMAGE_NAME} .'
            }
        }
         //    PUSH DOCKER IMAGE TO DOCKERHUB
        stage('Push Docker Image To DockerHub'){
            steps {
                script{
                    docker.withRegistry('', REGISTRY_CREDS ){
                        
                        sh "docker push ${IMAGE_NAME}"
                        sh "docker push latest"
                    }
                }
            }
        } 
        
        
        stage('Push Docker Image To JCR') {
            steps {
                sh "docker login -u cadmin -p P@ssw0rd http://192.168.1.3:8081/artifactory/docker_jfrog_repo/"
                sh "docker build -f Dockerfile . -t  192.168.1.3:8081/docker_jfrog_repo/${IMAGE_NAME}:${BUILD_NUMBER}"
                sh "docker push 192.168.1.3:8081/docker_jfrog_repo/${IMAGE_NAME}:${BUILD_NUMBER}"
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
                         sh "git push https://${user}:${pass}@github.com/YasserAhmedMoh/Devops_demo.git "
                    }
                }
            }
        }
        stage('Argocd login'){
             agent {label 'argocd_env'}
            steps {
                sh "git clone -b test https://github.com/YasserAhmedMoh/Devops_demo.git"
                sh "argocd login --username admin --insecure --password P@ssw0rd localhost:8090"
                sh "kubectl apply -f deployment.yml"
                
            }
        }
    }

post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#elprof',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} with the name ${env.IMAGE_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}


