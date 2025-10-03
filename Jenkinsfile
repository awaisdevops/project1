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

                    // Securely retrieves credentials using the 'github-credentials' ID.
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                                        
                        // --- FIX FOR URL ENCODING ISSUE ---
                        // The 'fatal: No such URL found' error occurs because Git URLs cannot contain unencoded spaces 
                        // in the username. We must URL-encode both the user and password/token.
                        def encodedUser = java.net.URLEncoder.encode(USER, 'UTF-8')
                        def encodedPass = java.net.URLEncoder.encode(PASS, 'UTF-8')
                        
                        // Construct the full, encoded remote URL
                        def remoteUrl = "https://${encodedUser}:${encodedPass}@github.com/awaisdevops/project1.git"
                        
                        // ------------------------------------

                        // Git configuration for commit author
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'

                        // Diagnostic commands (helpful for debugging)
                        sh 'git status'
                        sh 'git branch'
                        sh 'git config --list'
                        
                        // Set the remote URL using the URL-encoded credentials
                        // This command securely injects the credentials for subsequent push operations.
                        sh "git remote set-url origin ${remoteUrl}"
                        
                        // Perform the commit and push
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        // Push to the specific branch 'jenkins-jobs'
                        sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }
        
    }
}