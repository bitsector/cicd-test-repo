pipeline {

    environment {
        dockerimagename = "antonbiz/jenkins-test-app"
        dockerImage = ""
    }

    agent any

    stages {

        stage('Checkout Source') {
            steps {
                git 'https://github.com/bitsector/cicd-test-repo.git'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    dockerImage = docker.build(dockerimagename)
                }
            }
        }

        stage('Pushing Image') {
            environment {
                registryCredential = 'antonbiz-dockerhub'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                        dockerImage.push("1.0")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            parallel {
                stage('Deploy to GKE') {
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            step([
                                $class: 'KubernetesEngineBuilder',
                                projectId: 'gcp-cloud-run-tests',
                                clusterName: 'my-cluster',
                                location: 'asia-south1',
                                manifestPattern: 'deployment.yaml',
                                credentialsId: 'gke-credentials',
                                verifyDeployments: true
                            ])
                        }
                    }
                }
                stage('Deploy to GKE 2') {
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            step([
                                $class: 'KubernetesEngineBuilder',
                                projectId: 'gcp-cloud-run-tests',
                                clusterName: 'my-cluster',
                                location: 'asia-south1',
                                manifestPattern: 'service.yaml',
                                credentialsId: 'gke-credentials',
                                verifyDeployments: true
                            ])
                        }
                    }
                }
            }
        }
    }
}
