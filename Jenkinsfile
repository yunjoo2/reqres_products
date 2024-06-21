pipeline {
    agent any

    environment {
        REGION = 'ap-northeast-1'
        EKS_API = 'https://D9727AE3A76FEEB02D97240F10514057.gr7.ap-northeast-1.eks.amazonaws.com'
        EKS_CLUSTER_NAME = 'user08-eks'
        EKS_JENKINS_CREDENTIAL_ID = 'Kubernetes-Cred'
        ECR_PATH = '879772956301.dkr.ecr.ap-northeast-2.amazonaws.com'
        ECR_IMAGE = 'user08-product'
        AWS_CREDENTIAL_ID = 'AWS-Cred'
    }
    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }
        stage('Maven Build') {
            steps {
                withMaven(maven: 'Maven') {
                    sh 'mvn package -DskipTests'
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    image = docker.build("${ECR_PATH}/${ECR_IMAGE}")                  
                } 
            }  
        }
        stage('Push to ECR') {
            steps {
                script {
                    docker.withRegistry("https://${ECR_PATH}", "ecr:${REGION}:${AWS_CREDENTIAL_ID}") {
                        image.push("v${env.BUILD_NUMBER}")
                    }
                }
            }
        }
        stage('CleanUp Images') {
            steps {
                sh """
                docker rmi ${ECR_PATH}/${ECR_IMAGE}:v$BUILD_NUMBER
                docker rmi ${ECR_PATH}/${ECR_IMAGE}:latest
                """
            }
        }
        stage('Deploy to k8s') {
            steps {
                script {
                        withKubeConfig([credentialsId: "${EKS_JENKINS_CREDENTIAL_ID}",
                                        serverUrl: "${EKS_API}",
                                        clusterName: "${EKS_CLUSTER_NAME}"]) {
                            sh "sed 's/latest/v${env.BUILD_ID}/g' kubernetes/deploy.yaml > output.yaml"
                            sh "cat output.yaml"
                            sh "kubectl apply -f output.yaml"
                            sh "kubectl apply -f kubernetes/service.yaml"
                            sh "rm output.yaml"
                        }
                }
            }
        } 
    }
}
