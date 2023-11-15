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
                    awsDockerLogin "$AWS_ECR_URI","$AWS_DEFAULT_REGION"
                }
            }
        }

        stage('Check Changed Directories') {
            steps {
                script {
                    env.changedDirs = sh(script: 'git diff --name-only HEAD^ HEAD | grep / | cut -d/ -f1 | sort -u', returnStdout: true).trim()
                    echo "Changed Directories: ${env.changedDirs}"
                }
            }
        }

        stage('Build and push Worker Service image...') {
            when {
                expression { env.changedDirs.contains('worker') }
            }
            steps {
                dir('worker') {
                    script {
                        buildImage "$AWS_ECR_URI/$VOTE_SERVICE_IMAGE_NAME"
                        dockerPush "$AWS_ECR_URI/$VOTE_SERVICE_IMAGE_NAME"
                        deleteLocalImage "$AWS_ECR_URI/$VOTE_SERVICE_IMAGE_NAME"
                        deleteUntaggedImage "voteservice"
                    }
                }
            }
        }

        stage('Build and push Vote Service image...') {
            when {
                expression { env.changedDirs.contains('vote') }
            }
            steps {
                dir('vote') {
                    script {
                        buildImage "$AWS_ECR_URI/$VOTE_SERVICE_IMAGE_NAME"
                        dockerPush "$AWS_ECR_URI/$VOTE_SERVICE_IMAGE_NAME"
                        deleteLocalImage "$AWS_ECR_URI/$VOTE_SERVICE_IMAGE_NAME"
                        deleteUntaggedImage "voteservice"
                    }
                }
            }
        }

        stage('Build and push Result Service image...') {
            when {
                expression { env.changedDirs.contains('result') }
            }
            steps {
                dir('result') {
                    script {
                        buildImage "$AWS_ECR_URI/$RESULT_SERVICE_IMAGE_NAME"
                        dockerPush "$AWS_ECR_URI/$RESULT_SERVICE_IMAGE_NAME"
                        deleteLocalImage "$AWS_ECR_URI/$RESULT_SERVICE_IMAGE_NAME"
                        deleteUntaggedImage "resultservice"
                    }
                }
            }
        }

        stage('Deploy to EKS cluster...') {
            when {
                expression { env.changedDirs.contains('k8s-specifications') }
            }
            steps {
                script {
                    sh 'kubectl apply -f k8s-specifications/'
                }
            }
        }
        
        stage('Deploy worker service to EKS...') {
            when {
                expression { env.changedDirs.contains('worker') }
            }
            steps {
                sh "kubectl set image deployment/workerservice workerservice=$AWS_ECR_URI/$WORKER_SERVICE_IMAGE_NAME:dummy --record"
            }
        }

        stage('Deploy vote service to EKS...') {
            when {
                expression { env.changedDirs.contains('vote') }
            }
            steps {
                sh "kubectl set image deployment/voteservice voteservice=$AWS_ECR_URI/$VOTE_SERVICE_IMAGE_NAME:dummy --record"
            }
        }

        stage('Deploy result service to EKS...') {
            when {
                expression { env.changedDirs.contains('result') }
            }
            steps {
                sh "kubectl set image deployment/resultservice resultservice=$AWS_ECR_URI/$RESULT_SERVICE_IMAGE_NAME:dummy --record"
            }
        }
        
    }
}
