pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                image 'node:18-alpine'
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
                '''
                
            }
        }
        stage('Test') {
            agent {
                docker {
                image 'node:18-alpine'
                reuseNode true
                }
            }
            steps {
                sh '''
                npm test
                '''
            }
        }
        stage('Deploy') {
            agent {
                docker {
                image 'node:18-alpine'
                reuseNode true
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
            junit 'test-results/junit.xml'
        }
    }
    
}
