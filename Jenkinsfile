pipeline {
    agent any
    parameters {
        booleanParam(name: 'PUSH_IMAGE', defaultValue: true, description: 'Push Docker image to registry')
        booleanParam(name: 'DEPLOY', defaultValue: true, description: 'Deploy to Kubernetes cluster')
    }
    environment {
        REGISTRY = 'docker.io'
        IMAGE_NAME = 'myorg/cloud_etp'
        DOCKER_CREDENTIALS = 'dockerhub'
        KUBECONFIG_CREDENTIAL = 'kubeconfig_file'
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
                script {
                    def tag = "${IMAGE_NAME}:${env.BUILD_NUMBER}"
                    if (isUnix()) {
                        sh "docker build -t ${tag} ."
                    } else {
                        bat "docker build -t ${tag} ."
                    }
                    
                    if (params.PUSH_IMAGE) {
                        withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            if (isUnix()) {
                                sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin ${REGISTRY}"
                                sh "docker push ${tag}"
                            } else {
                                bat "echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin ${REGISTRY}"
                                bat "docker push ${tag}"
                            }
                        }
                    } else {
                        echo "Skipping image push (PUSH_IMAGE=false)"
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            when {
                expression { params.DEPLOY }
            }
            steps {
                withCredentials([file(credentialsId: env.KUBECONFIG_CREDENTIAL, variable: 'KUBECONFIG_FILE')]) {
                    script {
                        def tag = "${IMAGE_NAME}:${env.BUILD_NUMBER}"
                        if (isUnix()) {
                            sh "export KUBECONFIG=$KUBECONFIG_FILE"
                            sh "kubectl set image deployment/cloud-etp cloud-etp=${tag} --record || kubectl apply -f k8s/"
                        } else {
                            // On Windows, use PowerShell to set KUBECONFIG env var and run kubectl
                            powershell """
                                \$env:KUBECONFIG = \$env:KUBECONFIG_FILE
                                kubectl set image deployment/cloud-etp cloud-etp=${tag} --record
                                if (\$LASTEXITCODE -ne 0) {
                                    kubectl apply -f k8s/
                                }
                            """
                        }
                    }
                }
            }
        }
    }
}

