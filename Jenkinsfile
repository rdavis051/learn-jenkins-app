pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine' // Use Node.js 18 Alpine image
                    reuseNode true // Reuse the node for subsequent stages
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
        
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine' // Use Node.js 18 Alpine image
                    reuseNode true // Reuse the node for subsequent stages
                }
            }
            steps {
                echo 'Test stage...'
                sh '''
                    test -f build/index.html || (echo "build/index.html not found" && exit 1)
                    echo "build/index.html exists"
                    ls -l build/index.html
                    npm test
                '''
            }
        }

        stage('End-to-End Test') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy' // Use Playwright image
                    reuseNode true // Reuse the node for subsequent stages
                }
            }
            steps {
                echo 'Test stage...'
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build & sleep 10
                    npx playwright test --config=playwright.config.js --reporter=html
                '''
            }
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine' // Use Node.js 18 Alpine image
                    reuseNode true // Reuse the node for subsequent stages
                }
            }
            steps {
                sh '''
                    npm install netlify-cli -g
                    netlify --version
                '''
            }
        }
    }
    post {
        always {
            junit 'jest-results/junit.xml' // Adjust path as necessary
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
        success {
            echo 'Build and test succeeded!'
        }
        failure {
            echo 'Build or test failed!'
        }
    }
}