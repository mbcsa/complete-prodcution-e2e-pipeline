pipeline{
    agent {
        label "devops-agent"
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
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
                withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                    sh "mvn sonar:sonar"
                }
            }
        }
    }
}
