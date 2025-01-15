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
                stage('K8s Deployment') {
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            withCredentials([file(credentialsId: 'k8s.yaml', variable: 'KUBECONFIG_FILE')]) {
                                sh '''
                                    export KUBECONFIG=$KUBECONFIG_FILE
                                    kubectl get nodes
                                    kubectl apply -f deployment.yaml --validate=false
                                    kubectl apply -f service.yaml --validate=false
                                '''
                            }
                        }
                    }
                }
                stage('Deploy to GKE') {
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            step([
                                $class: 'KubernetesEngineBuilder',
                                projectId: 'gcp-cloud-run-tests',
                                clusterName: 'my-cluster',
                                location: 'us-central1-a',
                                manifestPattern: 'deployment.yaml,service.yaml',
                                credentialsId: 'gke-credentials',
                                verifyDeployments: true
                            ])
                        }
                    }
                }
                stage('Independent Stage 1') {
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            sh 'echo "hello world stage 1" > text1.txt'
                        }
                    }
                }
                stage('Independent Stage 2') {
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            sh 'echo "hello world stage 2" > text2.txt'
                        }
                    }
                }
            }
        }
    }
}
