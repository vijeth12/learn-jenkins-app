pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-v $WORKSPACE:/app -w /app'
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
                    ls -la build
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-v $WORKSPACE:/app -w /app'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Running tests..."
                    npm test
                '''
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    args '-v $WORKSPACE:/app -w /app'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Contents of build folder:"
                    ls -la build

                    npm install serve
                    chmod +x node_modules/.bin/serve

                    # Start the app in the background
                    node_modules/.bin/serve -s build -l 3000 &
                    sleep 10

                    npx playwright test --reporter=junit --output=playwright-report
                '''
            }
        }
    }

    post {
        always {
            junit 'playwright-report/results.xml'
        }
    }
}
