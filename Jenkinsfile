pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '33ec405f-7568-4d64-bad3-ed35a0176b38'
        NETLIFY_AUTH_TOKEN = credentials('netlify-id')

    }

    stages {
        /*
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
                   ls -la
                '''
            }
        }
        */
        stage('Docker'){
            steps{
                sh 'docker build -t my-playwright .'
            }
        }
        stage ("Test") {
            parallel {
                stage("Unit Test") {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                
                    steps {
                        sh '''
                        echo 'Test Stage....'
                        test -f build/index.html
                        npm test
                        '''
                    }
                
                    post {
                         always {
                        junit 'jest-results/junit.xml'
                         }
                    }
                }
                stage("E2E Test") {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        echo 'E2E Test Stage....'
                        serve -s build &
                        sleep 10
                        npx playwright test --reporter=html
                        '''
                    }
                
                
                    post {
                        always {
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                
                }

            }
        }
        stage("Approval"){
            steps{
                echo "Waiting for approval"
                timeout(time:5, unit:'MINUTES'){
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }

            }
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                  netlify --version
                  echo "The Site Id : $NETLIFY_SITE_ID"
                  netlify status
                  netlify deploy --dir build --prod

                '''
            }
        }
        
    }

}
