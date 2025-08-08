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
    }
}