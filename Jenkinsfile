pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'b3f524b2-3255-4e7b-9df7-8135f77f5d2e'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        /* ---------------------- BUILD ---------------------- */

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "--- Node Info ---"
                    node -v
                    npm -v

                    echo "--- Installing Dependencies ---"
                    npm ci

                    echo "--- Running Build ---"
                    npm run build
                '''
            }
        }

        /* ---------------------- TESTS ---------------------- */

        stage('Tests') {
            parallel {

                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "--- Running Unit Tests ---"
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "--- Installing Serve ---"
                            npm init -y >/dev/null 2>&1 || true
                            npm install serve

                            echo "--- Starting Local Server ---"
                            node_modules/.bin/serve -s build -l 3000 &

                            echo "--- Waiting for Server ---"
                            for i in {1..10}; do
                              nc -z localhost 3000 && echo "Server Ready" && break
                              echo "Waiting..."
                              sleep 2
                            done

                            echo "--- Running Playwright Tests ---"
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright HTML Report'
                            ])
                        }
                    }
                }
            }
        }

        /* ---------------------- DEPLOY ---------------------- */

        stage('Deploy to Netlify') {
            agent {
                docker {
                    image 'node:18'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "--- Installing Netlify CLI ---"
                    npm install netlify-cli

                    echo "--- Checking Build Directory ---"
                    test -d build || (echo "ERROR: build directory missing!" && exit 1)

                    echo "--- Deploying to Netlify ---"
                    node_modules/.bin/netlify deploy --dir=build --prod

                    echo "🎉 Deployment Successful!"
                '''
            }
        }
    }
}
