pipeline {
    agent {
        label "jenkins-Agent"
    }
    tools {
        jdk 'java17'
        maven 'maven3'
    }

    environment {
        APP_NAME = "complete-prodcution-e2e-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "hajar11980"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/hajar-elmader/complete-prodcution-e2e-pipeline.git'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        docker.withRegistry('', DOCKER_USER) {
                            def docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                            docker_image.push()
                            docker_image.push('latest')
                        }
                    }
                }
            }
        }
    }
}
