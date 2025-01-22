pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID = '5455f9bf-dea1-4e7e-8b45-7525172c0f7d'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        
        stage('aws'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
            }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'aws-cli', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) { 
                    sh'''
                    aws --version
                    aws s3 ls
                    '''
               }
                
            }
        }
    
        
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
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps{
                        sh'''
                        serve -s build &
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
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
                }
            steps{
                sh'''
                    echo "deploy to non-production site id: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > json-output.json
                    sleep 10
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' json-output.json) 
                    npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright post staging Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }
        }

        stage('approval'){
            steps{
                input 'would you like to deploy?'
            }

        }
        stage('Deploy E2E Test'){
            agent{
                docker{
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'https://chic-gingersnap-878078.netlify.app'
                }
            steps{
                sh'''
                    echo "deploy to production to production site id: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod 
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