import groovy.json.JsonSlurper

pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                    sh 'git fetch --tags'
                    def commitMessage = sh(script: "git log -1 --pretty=%B", returnStdout: true).trim()
                    if (commitMessage.contains("[skip ci]") || commitMessage.contains("k8s/")) {
                        error("Commit message contains '[skip ci]' or changes made in 'k8s/', skipping pipeline run")
                    }
                    def commitID = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    def tag = sh(script: "git tag --contains ${commitID}", returnStdout: true).trim()
                    env.GIT_TAG_NAME = tag ? tag : commitID.substring(0, 7)
                }
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: "aws-access",
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                        sh 'docker build -t 955865924758.dkr.ecr.eu-central-1.amazonaws.com/pythontestapp:${env.GIT_TAG_NAME} .'
                        sh 'aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin 955865924758.dkr.ecr.eu-central-1.amazonaws.com'
                        sh 'docker push 955865924758.dkr.ecr.eu-central-1.amazonaws.com/pythontestapp:${env.GIT_TAG_NAME}'
                    }
                }
            }
        }
        stage('Commit and Push Changes') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-s', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh '''
                        git config --global user.email 'saifaliunity@gmail.com'
                        git config --global user.name 'Saif Ali'
                        git add k8s/manifest.yaml
                        git commit -m "Update image tag to ${env.GIT_TAG_NAME}"
                        git push origin main
                    '''
                }
            }
        }
    }
} 
