@Library('jenkins-shared-library')_

pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'eu-west-3'
        WORKER_SERVICE_IMAGE_NAME = 'workerservice:latest'
        VOTE_SERVICE_IMAGE_NAME = 'voteservice:latest'
        RESULT_SERVICE_IMAGE_NAME = 'resultservice:latest'
        EKS_CLUSTER_NAME = ''
        AWS_ECR_URI = '733141109198.dkr.ecr.eu-west-3.amazonaws.com/resultservice'
    }

    stages {
        
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                echo "testing the app...."
            }
        }

        stage('Build and push image') {
            steps {
                dir('worker') {
                    script {
                        buildImage "$AWS_ECR_URI/$WORKER_SERVICE_IMAGE_NAME"
                        dockerPush "$AWS_ECR_URI/$WORKER_SERVICE_IMAGE_NAME"
                        deleteImage "$AWS_ECR_URI/$WORKER_SERVICE_IMAGE_NAME"
                    }
                }
                dir('vote') {
                    script {
                        buildImage "$AWS_ECR_URI/$VOTE_SERVICE_IMAGE_NAME"
                        dockerPush "$AWS_ECR_URI/$VOTE_SERVICE_IMAGE_NAME"
                        deleteImage "$AWS_ECR_URI/$VOTE_SERVICE_IMAGE_NAME"
                    }
                }
                dir('result') {
                    script {
                        buildImage "$AWS_ECR_URI/$RESULT_SERVICE_IMAGE_NAME"
                        dockerPush "$AWS_ECR_URI/$RESULT_SERVICE_IMAGE_NAME"
                        deleteImage "$AWS_ECR_URI/$RESULT_SERVICE_IMAGE_NAME"
                    }
                }
                
            }
        }
        
        stage('Deploy') {
            steps {
                echo "deploying the app...."
            }
        }
        
    }
}
