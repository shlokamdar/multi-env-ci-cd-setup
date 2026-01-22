pipeline {
    agent none

    options {
        skipDefaultCheckout(true)
    }

    environment {
        DEPLOY_DIR = '/usr/share/nginx/html'
    }

    stages {

        stage('Checkout') {
            agent any
            steps {
                checkout scm

                script {
                    // Reliable branch detection (detached HEAD safe)
                    if (env.BRANCH_NAME) {
                        env.ACTUAL_BRANCH = env.BRANCH_NAME
                    } else if (env.GIT_BRANCH) {
                        env.ACTUAL_BRANCH = env.GIT_BRANCH.replace('origin/', '')
                    } else {
                        error('Unable to determine branch name')
                    }
                }

                echo "Checked out branch: ${env.ACTUAL_BRANCH}"
            }
        }

        stage('Deploy to UAT') {
            when {
                expression { env.ACTUAL_BRANCH == 'uat' }
            }
            agent { label 'uat-slave' }

            stages {
                stage('Install Dependencies') {
                    steps {
                        echo 'Deploying to UAT'
                        sh 'npm ci'
                    }
                }

                stage('Build') {
                    steps {
                        sh 'npm run build'
                    }
                }

                stage('Deploy') {
                    steps {
                        sh """
                          sudo rm -rf ${DEPLOY_DIR}/*
                          sudo cp -r dist/* ${DEPLOY_DIR}/
                        """
                        echo 'UAT deployment completed'
                    }
                }
            }
        }

        stage('Deploy to PROD') {
            when {
                expression { env.ACTUAL_BRANCH == 'main' }
            }
            agent { label 'prod-slave' }

            stages {
                stage('Install Dependencies') {
                    steps {
                        echo 'Deploying to PRODUCTION'
                        sh 'npm ci'
                    }
                }

                stage('Build') {
                    steps {
                        sh 'npm run build'
                    }
                }

                stage('Deploy') {
                    steps {
                        sh """
                          sudo rm -rf ${DEPLOY_DIR}/*
                          sudo cp -r dist/* ${DEPLOY_DIR}/
                        """
                        echo 'Production deployment completed'
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline finished successfully for branch: ${env.ACTUAL_BRANCH}"
        }
        failure {
            echo "Pipeline failed for branch: ${env.ACTUAL_BRANCH}"
        }
    }
}
