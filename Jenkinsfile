pipeline {
    agent any

    environment { 
        REACT_APP_VERSION = "1.0.${BUILD_ID}" // Set the version dynamically based on the build number
        AWS_DEFAULT_REGION = 'us-east-1' // Set your AWS region
        AWS_ECS_CLUSTER = 'LearnJenkinsApp-Cluster-Prod' // Specify your ECS cluster name
        AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-TaskDefinition-Prod-service-2veot5a5' // Specify your ECS service name
        AWS_ECS_TD_PROD = 'LearnJenkinsApp-TaskDefinition-Prod' // Specify your ECS task definition name
    }

    stages {

        stage('Build') {
            // This stage builds the application using Node.js in a Docker container
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Building the application"
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
            // This stage builds the Docker image for the application
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    // Use the AWS CLI image to interact with AWS services
                    // Use the --entrypoint='' to avoid running the default entrypoint of the image
                    // This allows us to run the AWS CLI commands directly
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }
            steps {
                sh '''
                    echo "Building Docker image"
                    yum install -y docker
                    docker build -t myjenkinsapp .
                    docker images
                '''
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    // Use the AWS CLI image to interact with AWS services
                    // Use the --entrypoint='' to avoid running the default entrypoint of the image
                    // This allows us to run the AWS CLI commands directly
                    args "--entrypoint=''"
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-s3', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        echo "Connecting to AWS via AWS CLI"
                        aws --version
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo $LATEST_TD_REVISION
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD_PROD:${LATEST_TD_REVISION}
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD
                    '''                    
                }
            }
        }
    }
}