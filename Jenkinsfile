def imageName="192.168.44.44:8082/docker_registry/frontend"
def dockerRegistry="https://192.168.44.44:8082"
def registryCredentials="artifactory"
def dockerTag=""

pipeline {
    agent any
    
    environment {
        scannerHome = tool 'SonarQube'
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
        stage('pull backend') {
            steps {
                git url: 'https://github.com/mdukat/FrontendPanda.git', branch: 'main'
            }
        }
        stage('unit test') {
            steps {
                sh "pip3 install -r requirements.txt"
                sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml"
            }
        }
        stage('sonar') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage('build') {
            steps {
                script {
                    dockerTag = "${env.BUILD_ID}"
                    applicationImage = docker.build("$imageName:$dockerTag", ".")
                }
            }
        }
        stage('push') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }
            }
        }
    }
    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }
}
