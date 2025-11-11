pipeline {
    agent any

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Build') {
    agent {
        docker {
            image 'node:18-alpine'
            reuseNode true
            args '--user root'
        }
    }
    steps {
        sh '''
            ls -la
            node --version
            npm --version
            # Clean both node_modules and package-lock.json, then use npm install
            rm -rf node_modules
            rm -f package-lock.json
            npm install
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
                            args '--user root'  // Run as root to avoid permission issues
                        }
                    }

                    steps {
                        sh '''
                            # Clean any existing test results
                            rm -rf jest-results
                            mkdir -p jest-results
                            
                            # Install jest-junit if not already present
                            npm list jest-junit || npm install --save-dev jest-junit
                            
                            # Run tests with JUnit reporter
                            npm test -- --ci --silent --watchAll=false --testResultsProcessor=jest-junit --reporters=default --reporters=jest-junit
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            args '--ipc=host --user root'  // Run as root to avoid permission issues
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            # Install serve if not present
                            npm list serve || npm install serve
                            
                            # Start the server in background
                            node_modules/.bin/serve -s build -p 3000 &
                            SERVE_PID=$!
                            echo "Server started with PID: $SERVE_PID"
                            
                            # Wait for server to start
                            sleep 10
                            
                            # Test if server is responding
                            curl -f http://localhost:3000
                            
                            # Run Playwright tests
                            npx playwright test --reporter=html
                            
                            # Kill the server
                            kill $SERVE_PID
                        '''
                    }

                    post {
                        always {
                            archiveArtifacts artifacts: 'playwright-report/**/*'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
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
                    args '--user root'  // Run as root to avoid permission issues
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                '''
            }
        }
    }
    
    post {
        always {
            echo "Pipeline completed - check test results above"
            archiveArtifacts artifacts: 'build/**/*,jest-results/**/*,playwright-report/**/*'
        }
        failure {
            echo "Pipeline failed! Check the test results above. ❌"
        }
        success {
            echo "Pipeline succeeded! ✅"
        }
    }
}