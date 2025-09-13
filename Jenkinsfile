pipeline {
    agent any
    environment {
        REACT_APP_VERSION = "1.0.${BUILD_ID}"
        AWS_DEFAULT_REGION = 'us-east-1'
        AWS_ECS_CLUSTER = 'LearnJenkinsApp-cluster-prod'
        AWS_ECS_SERVICE = 'LearnJenkinsApp-Service-Prod'
        AWS_ECS_TASK_DEFINITION = 'LearnJenkinsApp-TaskDefinition-Prod'
    }
    stages {

         stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Build Docker Image') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }
            steps {

                sh '''
                    yum install -y amazon-linux-extras
                    amazon-linux-extras install docker
                    docker build -t myjenkinsapp -f Dockerfile .
                '''
                
               
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
    
               sh '''
                   echo  aws --version
                   yum  install jq -y
                   LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq -r '.taskDefinition.revision')
                   echo "Latest task definition revision: $LATEST_TD_REVISION"
                   aws ecs update-service --cluster $AWS_ECS_CLUSTER  --service $AWS_ECS_SERVICE  --task-definition $AWS_ECS_TASK_DEFINITION:$LATEST_TD_REVISION
                   aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                '''
                }
            
            }
        }


 



    }
}
