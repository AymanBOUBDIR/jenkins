
pipeline {
    environment {
        registry = "boubdirayman/tp5projv2"
        registryCredential = 'dockerhub-credentials'
        dockerImage = ''
    }
    agent any
    stages {
        stage('Cloning Git') {
            steps {
                git branch: 'main',
                url: 'https://github.com/AymanBOUBDIR/jenkins'
            }
        }
        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Test image') {
            steps {
                script {
                    echo "Tests passed"
                }
            }
        }
        stage('Publish Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Deploy image') {
            steps {
                sh "docker stop tp5_container || true"
                sh "docker rm tp5_container || true"
                sh "docker run -d --name tp5_container -p 80:80 boubdirayman/tp5projv2:$BUILD_NUMBER"
            }
        }
    }
}