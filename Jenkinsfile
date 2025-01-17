pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID = '90c7a9e6-900f-430b-8c48-6defdd8bc3cc'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh'''
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                ls -la
                '''
            }
        }
        

        stage('Tests'){
            parallel{
                stage('Unit Tests'){
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{
                        sh'''
                        echo Testing the Node app!
                        test -f build/index.html
                        npm test
                        '''
                    }
                    post{
                        always{
                            junit 'jest-results/junit.xml'
                            }
                  } 

        }
                stage('E2E'){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps{
                        sh'''
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 10
                        npx playwright test --reporter=html
                        '''
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                            }
                        }
              }
        }

        }

        stage('Deploy'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh'''
                 npm install netlify-cli
                 node_modules/.bin/netlify --version
                 echo "deploy to production to production site id: $NETLIFY_SITE_ID"
                 node_modules/.bin/netlify status
                 node_modules/.bin/netlify deploy --dir=build --prod 
                '''
            }
        }
  
    }  
}