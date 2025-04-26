pipeline {
    agent {
        label "jenkins-agent"
    }

    tools {
        jdk "Java17"
        maven "Maven3"

    }
    environment {
        APP_NAME = "complete-prodcution-e2e-pipeline"
        RELEASE = "1.0"
        DOCKER_USER = "jeejy30"
        DOCKER_PASS = "dockerhub"
        IMAGE_NAME = "${DOCKER_USER}"+"/"+"${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
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
                sh '''
                    mvn clean package
                '''
            }
        }
        stage('Test Uplication') {
            steps {
                sh '''
                    mvn test
                '''
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script{
                    withSonarQubeEnv(credentialsId: "sonarqube-token"){
                        sh '''
                            mvn sonar:sonar
                            '''
                    }
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script{
                   waitForQualityGate abortPipeline: false, credentialsId: "sonarqube-token" 
                }
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script{
                   docker.withRegistry('', DOCKER_PASS){
                        docker_image = docker.build("${IMAGE_NAME}")
                   }
                   docker.withRegistry('', DOCKER_PASS){
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                   }
                }
            }
        }
        stage('Trigger CD Pipeline') {
            steps {
                withCredentials([string(credentialsId: 'jenkins-api-token', variable: 'TOKEN')]) {
                   sh (script:'''
                        set +x
                        curl -v -k --user admin:$TOKEN \
                            -X POST \
                            -H 'cache-control: no-cache' \
                            -H 'content-type: application/www-form-urlencoded' \
                            --data "IMAGE_TAG=$IMAGE_TAG" \
                            "http://jenkins.jeejy.org/job/GitOps-pipeline/buildWithParameters?token=gitops-token"
                    ''')
                }
            }
        }


    }
}

