pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'bbabadara/ci-cd-demo'
        IMAGE_TAG = 'latest'
        RENDER_SERVICE_ID = 'renderSecret'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Analyse SonarQube') {
            steps {
                withSonarQubeEnv('SonarQubeLocal') {
                    script {
                        def scannerHome = tool 'SonarScanner'
                        bat """
                            ${scannerHome}\\bin\\sonar-scanner.bat ^
                            -Dsonar.projectKey=ci-cd-demo ^
                            -Dsonar.sources=. ^
                            -Dsonar.host.url=http://localhost:9000 ^
                            -Dsonar.login=%SonarSecret%
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-credentials') {
                        def image = docker.image("${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG}")
                        image.push()
                    }
                }
            }
        }

        stage('Deploy Render') {
            steps {
                bat """
                curl -X POST ^
                -H "Authorization: Bearer %render_api%" ^
                https://api.render.com/v1/services/%RENDER_SERVICE_ID%/deploys
                """
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline OK'
        }
        failure {
            echo '❌ Pipeline KO'
        }
    }
}