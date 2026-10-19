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
            parallel {
                stage ('Unit Tests') {
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

                    post {
                        always {
                            junit 'jest-results/*.xml'
                        }
                    }
                }
                
                stage ('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.crosoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 10
                        npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
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
                }
            }
            steps {
                sh '''
                 npm install netlify-cli
                 netlify --version   
                '''
            }
        }
    }

    post {
        always {

            junit 'test-results/*.xml'
           
        }
    }
}
