pipeline {
    agent any

    stages {
        stage('Docker') {
            parallel {
                stage('Docker - Playwright') {
                    steps {
                        sh 'docker build -f Dockerfile-my-playwright -t my-playwrite .'
                    }
                }

                stage('Docker - Node.js') {
                    steps {
                        sh 'docker build -f Dockerfile-my-node -t my-node .'
                    }
                }
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'my-node'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    node --version
                    npm --version
                    npm clean-install
                    npm run build
                '''
            }
        }

        stage('Test') {
            parallel {
                stage('Test - Unit') {
                    agent {
                        docker {
                            image 'my-node'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            test -f public/index.html
                            npm test
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('Test - E2E') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            serve -s build &
                            sleep 10 # Wait for serve to fully start
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML(
                                [
                                    allowMissing: false,
                                    alwaysLinkToLastBuild: false,
                                    keepAll: false,
                                    reportDir: 'playwright-report',
                                    reportFiles: 'index.html',
                                    reportName: 'Playwright HTML Report',
                                    reportTitles: '',
                                    useWrapperFileDirectly: true
                                ]
                            )
                        }
                    }
                }
            }
        }
    }
}
