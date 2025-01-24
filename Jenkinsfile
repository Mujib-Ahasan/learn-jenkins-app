pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID = '5455f9bf-dea1-4e7e-8b45-7525172c0f7d'
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

        stage('deploy to staging'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                    reuseNode true
            }
            }

            steps{
                withCredentials([usernamePassword(credentialsId: 'aws-cli', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) { 
                    sh'''
                    aws --version
                    aws s3 sync build s3://staging-for-jenkins-app --delete
                    '''
               }  
            }
        }        
        stage('E2E test after staging'){
            agent{
                docker{
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'http://staging-for-jenkins-app.s3-website-us-east-1.amazonaws.com'
                }
            steps{
                sh'''
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


        stage('deploy to production'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                    reuseNode true
            }
            }

            steps{
                withCredentials([usernamePassword(credentialsId: 'aws-cli', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) { 
                    sh'''
                    aws --version
                    aws s3 sync build s3://mujib
                    '''
               }  
            }
        }
        stage('E2E Test after deploy aws'){
            agent{
                docker{
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'http://mujib.s3-website-us-east-1.amazonaws.com'
                }
            steps{
                sh'''
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