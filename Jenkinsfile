pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                script {
                    echo 'Checking out code...'
                    git branch: 'main', url: 'https://github.com/Winson19/Stirling-PDF.git'
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    echo 'Building the project...'
                    sh 'chmod 755 gradlew'
                    sh './gradlew build'
                }
            }
            // Add post block to handle errors
            post {
                failure {
                    echo 'Build failed! Check the logs for more details.'
                    // Optionally notify via Slack or Email
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    echo 'Running tests...'
                    sh './gradlew test'
                }
                junit '**/build/test-results/test/*.xml'
            }
            post {
                failure {
                    echo 'Tests failed! Review the test results.'
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    echo 'Building Docker image...'
                    def appVersion = sh(returnStdout: true, script: './gradlew printVersion -q').trim()
                    def image = "frooodle/s-pdf:$appVersion"
                    sh "docker build -t $image ."
                }
            }
            post {
                failure {
                    echo 'Docker build failed!'
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    echo 'Pushing Docker image...'
                    def appVersion = sh(returnStdout: true, script: './gradlew printVersion -q').trim()
                    def image = "frooodle/s-pdf:$appVersion"
                    withCredentials([string(credentialsId: 'docker_hub_access_token', variable: 'DOCKER_HUB_ACCESS_TOKEN')]) {
                        sh "docker login --username frooodle --password $DOCKER_HUB_ACCESS_TOKEN"
                        sh "docker push $image"
                    }
                }
            }
            post {
                failure {
                    echo 'Docker push failed!'
                }
            }
        }
        stage('Helm Push') {
            steps {
                script {
                    echo 'Pushing Helm chart...'
                    // TODO: Read chartVersion from Chart.yaml
                    def chartVersion = '1.0.0'
                    withCredentials([string(credentialsId: 'docker_hub_access_token', variable: 'DOCKER_HUB_ACCESS_TOKEN')]) {
                        sh "docker login --username frooodle --password $DOCKER_HUB_ACCESS_TOKEN"
                        sh "helm package chart/stirling-pdf"
                        sh "helm push stirling-pdf-chart-$chartVersion.tgz oci://registry-1.docker.io/frooodle"
                    }
                }
            }
            post {
                failure {
                    echo 'Helm push failed!'
                }
            }
        }
    }
}
