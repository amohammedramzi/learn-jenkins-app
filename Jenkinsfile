pipeline {
    agent none  // No global agent
    
    stages {
        stage('Build & Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                sh '''
                    npm ci
                    npm run build
                    npm test
                '''
            }
        }
        
        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                sh '''
                    npm install -g netlify-cli
                    netlify --version
                    # Deploy commands
                '''
            }
        }
    }
    
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}