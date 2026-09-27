def imageName = "localhost:8082/docker_registry/backend"
def dockerRegistry = "http://localhost:8082"
def registryCredentials = "artifactory"
def dockerTag = ""

pipeline {
    agent {
        label 'docker'
    }
    options {
        skipDefaultCheckout()
    }
    environment {
        PIP_BREAK_SYSTEM_PACKAGES = 1
        scannerHome = tool 'SonarScanner'
    }
    stages {
        stage('Get Code') {
            steps {
                checkout scm
            }
        }
        stage('Unit tests') {
            steps {
                sh 'pip3 install -r requirements.txt'
                sh 'python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml'
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage('Build application image') {
            steps {
                script {
                    def shortCommit = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    dockerTag = "${shortCommit}-${env.BUILD_NUMBER}"
                    applicationImage = docker.build("$imageName:$dockerTag")
                }
            }
        }
        stage('Pushing image to docker registry') {
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
            junit testResults: 'test-results/pytest-report.xml', allowEmptyResults: true
            cleanWs()
        }
    }
}
