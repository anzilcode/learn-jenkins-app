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
                    test -f build/index.html
                    npm test
                '''
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build &
                    SERVER_PID=$!

                    sleep 10

                    # Generate JUnit XML
                    npx playwright test --reporter=junit

                    # Copy results into Jenkins workspace using PWD (not WORKSPACE)
                    mkdir -p $PWD/test-results-jenkins
                    cp -r test-results/* $PWD/test-results-jenkins/

                    kill $SERVER_PID
                '''
            }
        }
    }

    post {
        always {
            junit 'test-results-jenkins/**/*.xml'
        }
    }
}
