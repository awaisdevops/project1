pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment{
        SONAR_HOME= tool "SQ"
    }

    stages {

        stage('Prepare: Increment Application Version') {
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
        
        stage('CI: Compile and Package Application') {
            steps {
                script {
                    echo "building the application..."
                    // Use mvn clean package to compile, run tests, and package the app
                    // mvn builds app. clean removes older builds files. Dockerfile will fetch newer build app version 
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Test: Execute Unit Tests') {
            steps {
                echo 'Running Unit Tests...'
                // 'withMaven' step ensures the correct Maven environment is used
                withMaven(maven: 'maven') { 
                    sh 'mvn test'
                }
            }
            post {
                // 'always' ensures the reports are collected even if tests fail
                always {
                    // Collect and publish JUnit test reports
                    junit '**/target/surefire-reports/TEST-*.xml' 
                }
            }
        }

        stage('Test: Run Integration Tests') {
            steps {
                echo 'Running Integration Tests...'
                // Running 'verify' executes both the tests and the result check
                withMaven(maven: 'maven') { 
                    sh 'mvn verify' // This runs pre-integration-test, integration-test, and verify
                }
            }
            post {
                always {
                    // Collect and publish Failsafe test reports
                    junit '**/target/failsafe-reports/TEST-*.xml'
                }
            }
        }

        stage("Security & Quality: SonarQube Static Analysis"){
            steps{
                withSonarQubeEnv("SQ"){
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=dc-llc-app -Dsonar.projectKey=dc-llc-app"
                }
            }
        }

        stage("Security: OWASP Dependency Check (SCA)"){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/app-dep-check-report.html'
            }
        }

        stage("Security: Trivy Filesystem Scan"){
            steps{
                sh "trivy fs --format  table -o trivy-fs-report.html ."
            }
        }

        stage("Quality Gate: Wait for SonarQube Approval"){
            steps{
                timeout(time: 4, unit: "MINUTES"){
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        
        stage('Package: Build and Tag Docker Image') {

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

        stage('Security: Trivy Container Image Scan'){            
            steps{
                script {                    
                    // 1. Define local variable for the image tag
                    def FULL_IMAGE_TAG = "awaisakram11199/devopsimages:${env.IMAGE_NAME}"

                    // 2. Execute the shell command using the variable
                    sh "trivy image --format html -o trivy-image-report.html ${FULL_IMAGE_TAG}"
                    
                    // 3. Archive the report artifact (This step can be outside 'script' but is often placed here for flow)
                    archiveArtifacts artifacts: 'trivy-image-report.html', onlyIfSuccessful: true
                }
            }
        }

        stage('Deploy: Deploy to Environment') {

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

        stage('SCM: Commit New App Version') {
            steps {
                script {

                    // Retrieve the credentials. $PASS MUST be the GitHub Personal Access Token (PAT).
                  
                  withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                                  
                    // --- GITHUB PAT AUTH FIX ---
                    // GitHub rejects the traditional 'username:password@...' format.
                    // It requires the token to be used as the password with 'x-oauth-basic' as the placeholder username.
                    
                    def patUsername = "x-oauth-basic"
                    
                    // Construct the secure URL: https://x-oauth-basic:<PAT>@github.com/...
                    
                    def remoteUrl = "https://${patUsername}:${PASS}@github.com/awaisdevops/project1.git"
                    
                    // ---------------------------

                    // 1. Configure Git for the commit author                    
                    sh 'git config --global user.email "jenkins@example.com"'
                    sh 'git config --global user.name "jenkins"'

                    // 2. Set the remote URL using the PAT-based authentication URL                    
                    sh "git remote set-url origin ${remoteUrl}"
                    
                    // 3. Commit and Push                    
                    sh 'git add .'
                    sh 'git commit -m "Automated version bump  [skip ci]"'
                    sh 'git push origin HEAD:main'
                    }
                }
            }
        }
        
    }
}