pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID = '9a215b87-8ab7-4307-bc54-5968ba072fc6'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
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
                echo 'just changed a little bit!!!'
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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                            }
                        }
              }
        }

        }        
        stage('staging and E2E'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
                }
            steps{
                sh'''
                    npm install netlify-cli node-jq
                    echo "deploy to non-production site id: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > json-output.json
                    CI_ENVIRONMENT_URL= $(node_modules/.bin/node-jq -r '.deploy_url' json-output.json) 
                    npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright post staging Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }
        }
        stage('Deploy E2E Test'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'https://beautiful-cranachan-040876.netlify.app'
                }
            steps{
                sh'''
                    node --version
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "deploy to production to production site id: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod 
                    sleep 10
                    npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright post Deploy Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }
        }
  
    }  
}