pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '26ba2fee-af80-454a-9d71-5012e666e6e6'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
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
                    ls -la
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            #test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                // stage('E2E') {
                //     agent {
                //         docker {
                //             image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                //             reuseNode true
                //         }
                //     }

                //     steps {
                //         sh '''
                //             npm install serve
                //             node_modules/.bin/serve -s build &
                //             sleep 10
                //             npx playwright test  --reporter=html
                //         '''
                //     }

                //     post {
                //         always {
                //             publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright local', reportTitles: '', useWrapperFileDirectly: true])
                //         }
                //     }
                // }
            }
        }

        stage('Deploy Staging') {
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
                    echo "Deploying to staging with site id $NETLIFY_SITE_ID" 
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }

        stage('Approval') {
            steps {
                input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
            }
        }

        stage('Deploy Prod') {
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
                    echo "Deploying to prod with site id $NETLIFY_SITE_ID" 
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

//         stage('Prod E2E') {
//             agent {
//                 docker {
//                     image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
//                     reuseNode true
//                 }
//             }

//             environment {

//                        CI_ENVIRONMENT_URL = 'https://lovely-valkyrie-66a5b9.netlify.app'
                       
//                       }

//             steps {
//                 sh '''
//                     npx playwright test  --reporter=html
//                 '''
//             }

//             post {
//                 always {
//                     publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright e2e', reportTitles: '', useWrapperFileDirectly: true])
//                 }
//             }
// }
    }
}
