pipeline {
    agent any
    tools {
        maven 'maven'
    }
    stages {
        
        stage('build app') {
            steps {
                script {
                    echo "Building the application..."
                    // Use mvn clean package to compile, run tests, and package the app
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit and integration tests with Maven...'
                echo "Executing pipeline for branch $BRANCH_NAME"
                sh 'mvn test'
            }
        }
        
        stage('build image') {

            when {
                expression { 
                    BRANCH_NAME == 'master' 
                } 
            }

            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {

                        sh "docker build -t awaisakram11199/devopsimages:dcllcimg3 ."
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push awaisakram11199/devopsimages:dcllcimg3"
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