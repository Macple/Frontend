def imageName = 'xmacple/frontend'
def dockerTag = ''
def dockerRegistry = ''
def registryCredentials = 'dockerhub'

pipeline {
    agent {
        label 'agent'
    }
    
    environment {
        PIP_BREAK_SYSTEM_PACKAGES=1
        scannerHome = tool 'SonarQube' // the name you have given the Sonar Scanner (in Global Tool Configuration)
    }

    stages {
        stage('Get Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/Macple/Frontend'
            }
        }
        stage('Unit tests') {
            steps {
                    sh '''pip3 install -r requirements.txt
                        python3 -m pytest --cov=. --cov-report=xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml'''
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv(installationName: 'SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner -X"
                }
            }
        }
        stage('Build app image') {
            steps {
                script {
                    dockerTag = "RC-${env.BUILD_ID}"
                    // Budowanie obrazu z uwzględnieniem tagu
                    applicationImage = docker.build("${imageName}:${dockerTag}")
                }
            }
        }
        stage('Push image to the registry') {
            steps {
                script {
                    docker.withRegistry("${dockerRegistry}", "${registryCredentials}") {
                        // Wypchnij obraz z tagiem utworzonym w kroku budowania
                        applicationImage.push()
                        // Wypchnij obraz z tagiem 'latest'
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