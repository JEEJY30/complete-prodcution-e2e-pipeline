pipeline {
    agent {
        label "jenkins-agent"
    }

    tools {
        jdk "Java17"
        maven "Maven3"

    }

    stages {
        stage('Clean up workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from SCM') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/JEEJY30/complete-prodcution-e2e-pipeline' 
            }
        }
        stage('Build Application') {
            steps {
                sh "mvn clean package"
            }
        }
        stage('Test Uplication') {
            steps {
                sh "mvn test"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(credentialsId: "sonarqube-token"){
                    sh "mvn sonar:sonar"
                }
            }
        }
    }
}