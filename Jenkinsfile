pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQube'
        DOCKER_IMAGE = 'library-management:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/hoangmai29/library-management.git', branch: 'master'
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    bat 'mvn sonar:sonar'
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE% .'
            }
        }

        stage('Test PHP') {
            steps {
                bat '''
                    if exist login.php (
                        php -l login.php
                    ) else (
                        echo login.php not found. Skipping syntax check.
                    )
                '''
            }
        }

        stage('Deploy via SSH') {
            steps {
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'my-server',
                        transfers: [
                            sshTransfer(
                                sourceFiles: '**/*',
                                removePrefix: '',
                                remoteDirectory: '/home/user/library-app',
                                execCommand: 'docker restart library-container || docker run -d --name library-container -p 8080:8080 library-management:latest'
                            )
                        ],
                        verbose: true
                    )
                ])
            }
        }
    }

    post {
        failure {
            echo '❌ Pipeline failed!'
        }
        success {
            echo '✅ Pipeline completed successfully!'
        }
    }
}
