pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git-checkout') {
            steps {
                git 'https://github.com/jaiswaladi246/secretsanta-generator.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Tests') {
            steps {
                sh "mvn test"
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh """
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=santa \
                        -Dsonar.projectKey=santa \
                        -Dsonar.java.binaries=.
                    """
                }
            }
        }
        stage('Owasp Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan .', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build') {
            steps {
                sh "mvn package"
            }
        }
        stage('Docker Build Image') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker-cred', url: '']) {
                        sh "docker build -t santa:latest ."
                    }
                }
            }
        }
        stage('Tag & Push docker Image') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker-cred', url: '']) {
                        sh "docker tag santa:latest rajaveludocker/santa:latest"
                        sh "docker push rajaveludocker/santa:latest"
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker-cred', url: '']) {
                        sh "docker run -d -p 8081:8080 rajaveludocker/santa:latest"
                    }
                }
            }
        }
    }
}
