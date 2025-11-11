pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '--user root'  // Add this for permission issues
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

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                            args '--user root'
                        }
                    }
                    steps {
                        sh '''
                            # Create test results directory if it doesn't exist
                            mkdir -p jest-results
                            # Run tests with JUnit reporter
                            npm test -- --testResultsProcessor=\"jest-junit\" --ci --silent --reporters=default --reporters=jest-junit
                        '''
                    }
                    post {
                        always {
                            // Updated path for jest-junit results
                            junit 'jest-results/*.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            args '--user root --ipc=host'
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            # Start server in background and capture PID
                            node_modules/.bin/serve -s build -p 3000 &
                            SERVE_PID=$!
                            echo "Server started with PID: $SERVE_PID"
                            
                            # Wait for server to start
                            sleep 10
                            
                            # Check if server is running
                            curl -f http://localhost:3000 || echo "Server not ready"
                            
                            # Run Playwright tests
                            npx playwright test --reporter=html
                            
                            # Stop the server
                            kill $SERVE_PID || true
                        '''
                    }
                    post {
                        always {
                            // Archive the HTML report instead of using publishHTML
                            archiveArtifacts artifacts: 'playwright-report/**/*', allowEmptyArchive: true
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '--user root'
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deployment stage ready - add your deployment commands here"
                    # Example: netlify deploy --dir=build --prod
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline completed - check test results above"
            // Archive any test results for manual review
            archiveArtifacts artifacts: '**/test-results/**/*, **/jest-results/**/*, **/playwright-report/**/*', allowEmptyArchive: true
        }
        success {
            echo "Pipeline succeeded! 🎉"
        }
        failure {
            echo "Pipeline failed! Check the test results above. ❌"
        }
    }
}