pipeline {
    agent any

    environment { 
        NETLIFY_SITE_ID = '5676adf2-63ab-4059-8e1b-1fce7cf4b4da'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.${BUILD_ID}" // Set the version dynamically based on the build number
    }

    stages {

        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            environment {
                // Set the AWS credentials for the S3 bucket
                AWS_S3_BUCKET = 'learn-jenkins-202508132249'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-s3', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        echo "Installing AWS CLI"
                        aws --version
                        aws s3 ls
                        echo "<h1><c>Uploading file to S3!</c><h1>" > index.html
                        aws s3 cp index.html s3://$AWS_S3_BUCKET/index.html
                    '''                    
                }
            }
        }

        stage('Docker') {
            // This stage builds the Docker image for the application
            steps {
                sh '''
                    echo "Building Docker image"
                    docker build -t my-playwright .
                    docker images
                '''
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

        stage('Tests') {
            // This stage runs unit tests and end-to-end tests in parallel
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            #test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        // Publish the JUnit test results
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E tests') {
                    // This stage runs Playwright tests against the built application 
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test  --reporter=html
                        '''
                    }

                    post {
                        // Publish the Playwright HTML report
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy Staging') {
            // This stage deploys the built application to Netlify
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "STAGING_URL_TO_BE_SET" // Set the staging URL for the environment
            }

            steps {
                // Install Netlify CLI and deploy to staging. Removing the --prod flag to deploy to staging
                sh '''
                    netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                '''
                script {
                    // Extract the deploy URL from the JSON output
                    env.STAGING_URL = sh(script: "node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }   
            }
        }

        stage('Staging E2E') {
            // This stage runs Playwright tests against the built application in production
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment { 
                // Set the environment variable for the production URL
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }

            steps {
                // Run Playwright tests against the production URL
                sh '''
                    npx playwright test  --reporter=html
                '''
            }

            post {
                // Publish the Playwright HTML report
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Stageing E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Approval') {
            // This stage waits for manual approval before deploying to production
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure I want to deploy!'
                }
            }
        }

        stage('Deploy Prod') {
            // This stage deploys the built application to Netlify
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "https://neon-speculoos-456a4d.netlify.app" // Set the production URL for the environment
            }
            steps {
                sh '''
                    node --version
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E') {
            // This stage runs Playwright tests against the built application in production
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment { 
                // Set the environment variable for the production URL
                CI_ENVIRONMENT_URL = 'https://neon-speculoos-456a4d.netlify.app'
            }

            steps {
                // Run Playwright tests against the production URL
                sh '''
                    npx playwright test  --reporter=html
                '''
            }

            post {
                // Publish the Playwright HTML report
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}