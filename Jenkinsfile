pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID: '24cb3f9f-31f3-44d6-a2e8-f4887618529e'
    }

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
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test
                        '''
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
                 npm install netlify-cli@20.1.1
                 node_modules/.bin/netlify --version 
                 echo "Deploying to Netlify site ID: $NETLIFY_SITE_ID"
                '''
            }
        }
    }

    post {
        always {

            junit 'jest-results/*.xml'
           
        }
    }
    
}
