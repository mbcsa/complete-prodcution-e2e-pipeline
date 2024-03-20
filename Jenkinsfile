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
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        DOCKER_REGISTRY = "https://dkreg.corsisa.com.ar"
        DOCKER_REGISTRY_CREDENTIALS = "corsisa-registry-user"
        SONAR_CREDENTIALS = "jenkins-sonarqube-token"
        GIT_URL = "https://github.com/mbcsa/complete-prodcution-e2e-pipeline"
        GIT_CREDENTIALS = "github"
        GIT_BRANCH = "main"
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }
        stage("Checkout from SCM") {
            steps {
                git branch: GIT_BRANCH, credentialsId: GIT_CREDENTIALS, url: GIT_URL
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
                    withSonarQubeEnv(credentialsId: SONAR_CREDENTIALS) {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true, credentialsId: SONAR_CREDENTIALS
                }
            }
        }
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry(DOCKER_REGISTRY, DOCKER_REGISTRY_CREDENTIALS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }
                    /*docker.withRegistry(DOCKER_REGISTRY, DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                    docker.withRegistry(DOCKER_REGISTRY, DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }*/
                }
            }
        }
    }
}
