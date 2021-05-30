pipeline {
  environment {
    imagename = "sharran/goserverdemo"
    registryCredential = 'dockerhub_sharran'
    dockerImage = ''
  }
  agent any
  stages {
    stage('Build container') {
      steps{
        script {
          dockerImage = docker.build imagename
        }
      }
    }
    stage('Push container') {
      steps{
        script {
          docker.withRegistry('', registryCredential) {
            dockerImage.push("$BUILD_NUMBER")
            dockerImage.push("latest")}
        }
        sh "docker rmi $imagename:$BUILD_NUMBER"
        sh "docker rmi $imagename:latest"
      }
    }
    stage('Deploying to EKS') {
      steps {
        sh "curl -sS -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-07-26/bin/linux/amd64/aws-iam-authenticator"
        sh "curl -sS -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/kubectl"
        sh "chmod +x ./kubectl ./aws-iam-authenticator"
        sh "export PATH=$PWD/:$PATH"
        sh "apk update && apk add --no-cache python3 py3-pip && pip3 install --upgrade awscli"
        dir("k8s-config.d") {
          sh "kubectl apply -f goserver_deployment_service.yaml"
          sh "kubectl get all"
        }
      }
    }
  }
}
