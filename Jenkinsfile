pipeline {
    agent any
    tools {
        maven 'maven'
    }
    stages {

        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
        
        stage('build app') {
            steps {
                script {
                    echo "building the application..."
                    // Use mvn clean package to compile, run tests, and package the app
                    // mvn builds app. clean removes older builds files. Dockerfile will fetch newer build app version 
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('test app') {
            steps {
                echo 'running unit and integration tests with Maven...'
                //echo "Executing pipeline for branch $BRANCH_NAME"
                sh 'mvn test'
            }
        }
        
        stage('build image') {

            //when {
                //expression { 
                    //BRANCH_NAME == 'main' 
                //} 
            //}

            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {

                        sh "docker build -t awaisakram11199/devopsimages:${IMAGE_NAME} ."
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        sh "docker push awaisakram11199/devopsimages:${IMAGE_NAME}"
                        
                    }
                }
            }
        }
        stage('deploy') {

            //when {
                //expression { 
                    //BRANCH_NAME == 'main' 
                //} 
            //}
            
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                }
            }
        }

        stage('commit new app version') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        
                        # Git configuration (only needed once)
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'

                        sh 'git status'
                        sh 'git branch'
                        sh 'git config --list'
                        
                        # Push version bump to remote Git repository
                        sh "git remote set-url origin https://${USER}:${PASS}@github.com/awaisdevops/project1.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }
        
    }
}