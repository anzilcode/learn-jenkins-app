pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '81845997-e3f8-4d91-8a81-d78996d90297'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18-bullseye'
                }
            }
            steps {
                sh '''
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    test -d build
                '''
            }
        }

        stage('Tests') {
            parallel {

                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-bullseye'
                        }
                    }
                    steps {
                        sh '''
                            npm ci
                            npm test
                        '''
                    }
                }

                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        }
                    }
                    steps {
                        sh '''
                            npm ci
                            npx playwright install --with-deps
                            npx serve -s build &
                            sleep 10
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

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-bullseye'
                }
            }
            steps {
                sh '''
                    npm install -g netlify-cli
                    netlify --version
                    netlify status
                    netlify deploy --dir=build --prod --site=$NETLIFY_SITE_ID
                '''
            }
        }
    }
}
