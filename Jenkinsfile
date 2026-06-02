pipeline {
    agent any
    environment {
        REGISTRY = 'docker.io'
        IMAGE_NAME = 'myorg/cloud_etp'
        DOCKER_CREDENTIALS = 'dockerhub'
        KUBECONFIG_CREDENTIAL = 'kubeconfig-file'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'echo "Building app"'
                    } else {
                        bat 'echo Building app'
                    }
                }
            }
        }
        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        def tag = "${amvar0909/cloud_ETP}:${env.BUILD_NUMBER}"
                        if (isUnix()) {
                            sh "docker build -t ${tag} ."
                            sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin ${docker.io}"
                            sh "docker push ${tag}"
                        } else {
                            bat "docker build -t %IMAGE_NAME%:%BUILD_NUMBER% ."
                            bat "echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin %REGISTRY%"
                            bat "docker push %IMAGE_NAME%:%BUILD_NUMBER%"
                        }
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: env.KUBECONFIG_CREDENTIAL, variable: 'KUBECONFIG_FILE')]) {
                    script {
                        def tag = "${IMAGE_NAME}:${env.BUILD_NUMBER}"
                        if (isUnix()) {
                            sh 'mkdir -p $WORKSPACE/.kube'
                            sh 'cp $KUBECONFIG_FILE $WORKSPACE/.kube/config'
                            sh 'export KUBECONFIG=$WORKSPACE/.kube/config'
                            // try updating image, fallback to apply
                            sh "kubectl set image deployment/cloud-etp cloud-etp=${tag} --record || kubectl apply -f k8s/"
                        } else {
                            bat 'mkdir %WORKSPACE%\\.kube'
                            bat 'copy %KUBECONFIG_FILE% %WORKSPACE%\\.kube\\config'
                            bat 'set KUBECONFIG=%WORKSPACE%\\.kube\\config'
                            bat "kubectl set image deployment/cloud-etp cloud-etp=%IMAGE_NAME%:%BUILD_NUMBER% --record || kubectl apply -f k8s\\"
                        }
                    }
                }
            }
        }
    }
}

