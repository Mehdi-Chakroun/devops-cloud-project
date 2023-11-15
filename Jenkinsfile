@Library('jenkins-shared-library')_

pipeline {
    agent any
    environment {
        AWS_DEFAULT_REGION = 'eu-west-3'
        WORKER_SERVICE_IMAGE_NAME = 'workerservice:latest'
        VOTE_SERVICE_IMAGE_NAME = 'voteservice:latest'
        RESULT_SERVICE_IMAGE_NAME = 'resultservice:latest'
        EKS_CLUSTER_NAME = ''
        AWS_ECR_URI = '733141109198.dkr.ecr.eu-west-3.amazonaws.com'
    }

    stages {

        stage('Test') {
            steps {
                echo "testing the app...."
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    awsDockerLogin "AWS_ECR_URI","AWS_DEFAULT_REGION"
                }
            }
        }

        stage('Build and push image') {
            steps {
                dir('worker') {
                    script {
                        buildImage "$AWS_ECR_URI/$WORKER_SERVICE_IMAGE_NAME"
                        dockerPush "$AWS_ECR_URI/$WORKER_SERVICE_IMAGE_NAME"
                        deleteLocalImage "$AWS_ECR_URI/$WORKER_SERVICE_IMAGE_NAME"
                        echo "sleeping for 10 seconds"
                        sh 'sleep 10'
                        deleteUntaggedImage "workerservice"
                    }
                }
                dir('vote') {
                    script {
                        buildImage "$AWS_ECR_URI/$VOTE_SERVICE_IMAGE_NAME"
                        dockerPush "$AWS_ECR_URI/$VOTE_SERVICE_IMAGE_NAME"
                        deleteLocalImage "$AWS_ECR_URI/$VOTE_SERVICE_IMAGE_NAME"
                        echo "sleeping for 10 seconds"
                        sh 'sleep 10'
                        deleteUntaggedImage "voteservice"
                    }
                }
                dir('result') {
                    script {
                        buildImage "$AWS_ECR_URI/$RESULT_SERVICE_IMAGE_NAME"
                        dockerPush "$AWS_ECR_URI/$RESULT_SERVICE_IMAGE_NAME"
                        deleteLocalImage "$AWS_ECR_URI/$RESULT_SERVICE_IMAGE_NAME"
                        echo "sleeping for 10 seconds"
                        sh 'sleep 10'
                        deleteUntaggedImage "resultservice"
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
