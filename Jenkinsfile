pipeline {
    agent any
    
//    options {}

//    tools {}

    environment {
        AWS_ACCOUNT_ID="355100329777"
        AWS_DEFAULT_REGION="us-west-1"
        JENKINS_AWS_ID="my.aws.credentials"
        IMAGE_REPO_NAME="p3test"
        IMAGE_TAG="Latest"
        REPOSITORY_URL = "https://${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
        //DOCKER_CONTAINER_NAME = "P3-Backend-Dev"
        //AWSCLI_DIR = '/usr/local/bin/'
    }

    stages {
        stage('Git Clone') {
            steps {
                echo 'Git Cloning..'
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/piaohulin2022/p3test.git']]])
            }
        }
        stage(' Docker Image Building & push') {
            steps {
                echo 'Building image..'
                script {
                    docker.withRegistry("${REPOSITORY_URL}","ecr:${AWS_DEFAULT_REGION}:${JENKINS_AWS_ID}"){
                        def myImage = docker.build("${IMAGE_REPO_NAME}")
                        myImage.push("${IMAGE_TAG}")
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                echo 'ECS Deploying....'
                script{
                    try {
                        //withAWS(role: '{IAM Role 이름}', roleAccount: '{AWS 계정 번호}', externalId:'externalId') {
                        withAWS(region: 'us-west-1', credentials: 'my.aws.credentials') {
                            echo 'you are in here'
                            
                            sh 'aws ecs update-service --region us-west-1 --cluster p3test-ECS-Cluster --service p3test-ECS-Service --force-new-deployment'
                        }
                        
                        } catch (error) {
                            print(error)
                            echo 'Remove Deploy Files'
                            //sh "sudo rm -rf /var/lib/jenkins/workspace/${env.JOB_NAME}/*"
                            currentBuild.result = 'FAILURE'
                        }
                    }
                
                }
            }
        }
    
//    post {}
}