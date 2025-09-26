pipeline {
    agent any
    tools {
        maven 'maven'
    }
    stages {
        
        stage('build app') {
            steps {
                script {
                    echo "building the application..."
                    sh 'mvn clean package'
                }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', passwordVariable: 'PASS', usernameVariable: 'USER')]) {

                        sh "docker build -t awaisakram11199/demo-app:dc-llc-dev:1 ."
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push awaisakram11199/demo-app:dc-llc-dev:1"
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                }
            }
        }
        
    }
}