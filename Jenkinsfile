pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '7d812168-5fa9-4f75-a50e-4a3c2a8079aa'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '--user root'  // Run as root to avoid permission issues
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    # Clean node_modules to avoid permission issues
                    rm -rf node_modules
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
                            args '--user root'  // Run as root to avoid permission issues
                        }
                    }

                    steps {
                        sh '''
                            # Install jest-junit if not already present
                            npm list jest-junit || npm install --save-dev jest-junit
                            
                            # Run tests - they should automatically generate test-results/junit.xml
                            npm test -- --ci --watchAll=false
                            
                            # Check if the junit.xml file was created
                            ls -la test-results/ || echo "test-results directory not found"
                            ls -la test-results/junit.xml || echo "junit.xml file not found"
                        '''
                    }
                    post {
                        always {
                            // Update the path to match where your junit.xml is actually located
                            junit 'test-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            args '--ipc=host --user root'  // Add IPC host and run as root
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
                            // Use archiveArtifacts instead of publishHTML since the plugin is not installed
                            archiveArtifacts artifacts: 'playwright-report/**/*'
                            // Alternative: just archive the HTML file
                            archiveArtifacts artifacts: 'playwright-report/index.html'
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
                    echo "Deploying to Netlify site ID: $NETLIFY_SITE_ID"
                '''
            }
        }
    }
    
    post {
        always {
            echo "Pipeline completed - check test results above"
            // Archive all important artifacts
            archiveArtifacts artifacts: 'build/**/*,test-results/**/*,playwright-report/**/*'
        }
        failure {
            echo "Pipeline failed! Check the test results above. ❌"
        }
        success {
            echo "Pipeline succeeded! ✅"
        }
    }
}