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
        stage('Check Host') {
            steps {
                bat 'hostname'
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

                        powershell """
                            kubectl --kubeconfig=\"$env:KUBECONFIG_FILE\" get nodes

                            kubectl --kubeconfig=\"$env:KUBECONFIG_FILE\" set image deployment/cloud-etp cloud-etp=${tag}

                            if (\$LASTEXITCODE -ne 0) {
                                kubectl --kubeconfig=\"$env:KUBECONFIG_FILE\" apply -f k8s/
                            }
                        """
                    }
                }
            }
        }
    }
}