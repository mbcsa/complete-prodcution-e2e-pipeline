pipeline{
    agent {
        label "devops-agent"
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "complete-prodcution-e2e-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "csadocker"
        DOCKER_PASS = "Corsisa01"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        DOCKER_REGISTRY = "http://192.168.200.4"
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }
        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/mbcsa/complete-prodcution-e2e-pipeline'
            }
        }
        stage("Build") {
            steps {
                sh "mvn clean package"
            }
        }
        stage("Test") {
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
                    docker.withRegistry(DOCKER_REGISTRY, DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                    docker.withRegistry(DOCKER_REGISTRY, DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }
                }
            }
        }
    }
}
