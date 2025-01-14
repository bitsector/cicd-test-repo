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

    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }

    stage('Pushing Image') {
      environment {
               registryCredential = 'antonbiz-dockerhub'
           }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("1.0")
          }
        }
      }
    }

    // stage('Deploying flask app') {
    //   steps {
    //     script {
    //       kubernetesDeploy(configs: "deployment.yaml", "service.yaml", kubeconfigId: 'minikube')
    //     }
    //   }
    // }

    stage('Deploy') {
            steps {
                // Execute kubectl commands
                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl apply -f service.yaml'
            }
        }
  }

}