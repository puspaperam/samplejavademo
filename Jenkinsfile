pipeline {
    agent {
        label 'linux-docker-agent'
    }

    tools {
        git 'linux-git'
        maven 'apache-maven-3.9.9'
    }

    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Check Environment') {
            steps {
                sh 'git --version'
                sh 'docker --version'
            }
        }

        stage('Checkout the Code') {
            steps {
                git url: 'https://github.com/puspaperam/samplejavademo.git'
            }
        }

        stage('Build the Code') {
            steps {
                withMaven(maven: 'apache-maven-3.9.9') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('SonarQube Scan') {
            environment {
                scannerHome = tool 'sonarqube-scanner'
            }
            steps {
                withSonarQubeEnv('My SonarQube Server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build . -t puspaperam/samplejavademo:1.1.2'
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    def cid = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    echo "Commit ID: ${cid}"

                    withCredentials([string(credentialsId: 'docker-hub1', variable: 'hubpwd')]) {
                        sh "echo ${hubpwd} | docker login -u puspaperam --password-stdin"
                        sh "docker tag puspaperam/samplejavademo:1.1.2 puspaperam/samplejavademo:${cid}"
                        sh "docker push puspaperam/samplejavademo:${cid}"
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sshagent(['tomcat-credentials']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no target/hiring.war root@192.168.124.129:/opt/tomcat/webapps/
                        ssh -o StrictHostKeyChecking=no root@192.168.124.129 "/opt/tomcat/bin/shutdown.sh"
                        ssh -o StrictHostKeyChecking=no root@192.168.124.129 "/opt/tomcat/bin/startup.sh"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build completed successfully.'
        }
        failure {
            echo '❌ Build failed.'
        }
    }
}

       
 
