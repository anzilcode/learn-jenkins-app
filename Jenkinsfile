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
                    # install serve
                    npm install serve

                    # start server
                    node_modules/.bin/serve -s build &
                    SERVER_PID=$!

                    sleep 10

                    # run E2E with junit reporter
                    npx playwright test --reporter=junit

                    # ensure workspace has test results
                    mkdir -p ${WORKSPACE}/test-results
                    cp -r test-results/* ${WORKSPACE}/test-results/

                    kill $SERVER_PID
                '''
            }
        }
    }

    post {
        always {
            junit 'test-results/**/*.xml'
        }
    }
}

