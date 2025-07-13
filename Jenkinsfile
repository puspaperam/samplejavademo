pipeline {
    agent {
        label 'linuxdocker'
    }

    tools {
        git 'Default'
        maven 'apache-maven-3.9.9'
    }

    stages {
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

        stage('Docker Build') {
            steps {
                sh 'docker build . -t puspaperam/samplejavademo:1.1.2'
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    // Define closure inside script block
                    def commit_id = {
                        return sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    }

                    def cid = commit_id()
                    echo "Commit ID: ${cid}"

                    withCredentials([string(credentialsId: 'docker-hub1', variable: 'hubpwd')]) {
                        sh "echo ${hubpwd} | docker login -u puspaperam --password-stdin"
                        sh "docker tag puspaperam/samplejavademo:1.1.2 puspaperam/samplejavademo:${cid}"
                        sh "docker push puspaperam/samplejavademo:${cid}"
                    }
                }
            }
        }

      
   
