pipeline {
    agent any

    environment { 
        REACT_APP_VERSION = "1.0.${BUILD_ID}" // Set the version dynamically based on the build number
        AWS_DEFAULT_REGION = 'us-east-1' // Set your AWS region
    }

    stages {

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
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
                        aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json   
                        aws ecs update-service --cluster LearnJenkinsApp-Cluster-Prod --service LearnJenkinsApp-TaskDefinition-Prod-service-2veot5a5 --task-definition LearnJenkinsApp-TaskDefinition-Prod:2
                    '''                    
                }
            }
        }

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
    }
}