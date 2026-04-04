pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKERHUB_REPO        = 'emciptdevops'
        SONAR_PROJECT_KEY     = 'food-delivery-app'
        SONAR_HOST_URL        = 'http://3.105.183.152:9000'   
        COMPOSE_FILE          = 'docker-compose.yml'
    }

    stages {

        // ─────────────────────────────────────────────
        // STAGE 1 — Git Checkout
        // ─────────────────────────────────────────────
        stage('Git Checkout') {
            steps {
                echo '>>> Checking out source code...'
                checkout scm
            }
        }

        // ─────────────────────────────────────────────
        // STAGE 2 — Build Validation
        // Verify that the source code compiles / installs
        // correctly for all three services.
        // ─────────────────────────────────────────────
        stage('Build Validation') {
            parallel {
                stage('Validate Frontend') {
                    steps {
                        dir('frontend') {
                            echo '>>> Validating frontend dependencies...'
                            sh 'npm ci --prefer-offline || npm install'
                            sh 'npm run build'
                        }
                    }
                }
                stage('Validate Backend') {
                    steps {
                        dir('backend') {
                            echo '>>> Validating backend dependencies...'
                            sh 'npm ci --prefer-offline || npm install'
                        }
                    }
                }
                stage('Validate Admin') {
                    steps {
                        dir('admin') {
                            echo '>>> Validating admin dependencies...'
                            sh 'npm ci --prefer-offline || npm install'
                            sh 'npm run build'
                        }
                    }
                }
            }
        }

        // ─────────────────────────────────────────────
        // STAGE 3 — SonarQube Analysis
        // ─────────────────────────────────────────────
        stage('SonarQube Analysis') {
            steps {
                echo '>>> Running SonarQube code quality scan...'
                withSonarQubeEnv('SonarQube') {   
                    sh """
                        sonar-scanner \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.sources=frontend/src,backend,admin/src \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.token=${env.SONAR_AUTH_TOKEN}
                    """
                }
            }
        }

        // ─────────────────────────────────────────────
        // STAGE 4 — Docker Image Build
        // Uses docker-compose so all three images are
        // built with a single command.
        // ─────────────────────────────────────────────
        stage('Docker Image Build') {
            steps {
                echo '>>> Building Docker images via docker-compose...'
                sh 'docker compose -f ${COMPOSE_FILE} build'
            }
        }

        // ─────────────────────────────────────────────
        // STAGE 5 — Push Images to Docker Hub
        // ─────────────────────────────────────────────
        stage('Push to Docker Hub') {
            steps {
                echo '>>> Logging in to Docker Hub and pushing images...'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker compose -f ${COMPOSE_FILE} push'
            }
        }

        // ─────────────────────────────────────────────
        // STAGE 6 — Deploy (optional but recommended)
        // Pull fresh images and restart the stack on
        // the same EC2 instance.
        // ─────────────────────────────────────────────
        stage('Deploy') {
            steps {
                echo '>>> Deploying updated stack...'
                sh 'docker compose -f ${COMPOSE_FILE} pull'
                sh 'docker compose -f ${COMPOSE_FILE} up -d --remove-orphans'
            }
        }
    }

    // ─────────────────────────────────────────────────
    // POST — Notifications
    // ─────────────────────────────────────────────────
    post {
        success {
            echo '>>> Pipeline completed successfully!'
            mail(
                to:      'suryalogu616@gmail.com',        
                subject: "✅ BUILD SUCCESS — ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body:    """\
Hi Team,

The Jenkins pipeline for ${env.JOB_NAME} completed successfully.

Build Number : ${env.BUILD_NUMBER}
Build URL    : ${env.BUILD_URL}

Regards,
Jenkins
"""
            )
        }

        failure {
            echo '>>> Pipeline FAILED. Sending failure email...'
            mail(
                to:      'suryalogu616@gmail.com',        
                subject: "❌ BUILD FAILED — ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body:    """\
Hi Team,

The Jenkins pipeline for ${env.JOB_NAME} has FAILED.

Build Number : ${env.BUILD_NUMBER}
Build URL    : ${env.BUILD_URL}

Please check the Jenkins console output for details.

Regards,
Jenkins
"""
            )
        }

        always {
            echo '>>> Cleaning up Docker login session...'
            sh 'docker logout || true'
        }
    }
}
