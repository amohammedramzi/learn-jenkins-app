pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            // Runs as non-root 'node' user by default
        }
    }
    
    stages {
        stage('Build') {
            steps {
                sh '''
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
            post {
                always {
                    junit 'test-results/junit.xml'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                sh '''
                    # Install Netlify CLI locally instead of globally
                    npm install netlify-cli
                    npx netlify --version
                    # Add your actual Netlify deploy command here
                    # npx netlify deploy --prod --dir=build
                '''
            }
        }
    }
}