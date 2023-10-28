@Library('jenkins-shared-library')_

pipeline {
    agent any
    tools {
        maven 'maven-3.9.5'
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }

            }
        }
        stage('Test') {
            steps {
                echo "testing the app...."
                sh 'mvn test'
            }
        }
        stage('Build jar') {

            steps {
                script {
                    buildJar()
                }
            }
        }
        stage('Build and push image') {

            steps {
                script {
                    buildImage "schkoda/push-from-jenkins:$IMAGE_NAME"
                    dockerLogin()
                    dockerPush "schkoda/push-from-jenkins:$IMAGE_NAME"
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo "deploying the app...."
            }
        }
        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'Mehdi-GitHub', passwordVariable: 'GITHUB_PASSWORD', usernameVariable: 'GITHUB_USERNAME')]) {
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'
                        sh "git remote set-url origin https://${GITHUB_USERNAME}:${GITHUB_PASSWORD}@github.com/Mehdi-Chakroun/simple-java-maven-app"
                        sh 'git add .'
                        sh 'git commit -m "ci: version update"'
                        sh 'git push origin HEAD:jenkins-job'
                    }
                }
            }
        }
    }
}
