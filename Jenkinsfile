pipeline {
    agent any

    environment {
        PROJECT_ID = 'careful-ensign-470212-v5'
        GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp-service-account')
        DOCKER_HUB_USERNAME = 'dockernikhil999'
        IMAGE_NAME = 'cloudrun'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/nikhilvadla500/cloudrun-jenkins-2025.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // login + push inside same credentials block
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-password', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh '''
                            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                            docker push ${DOCKER_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }

        stage('Deploy to Google Cloud Run') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh """
                            gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                            gcloud config set project ${PROJECT_ID}
                            gcloud run deploy ${IMAGE_NAME} \
                                --image docker.io/${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER} \
                                --platform managed \
                                --region us-central1 \
                                --allow-unauthenticated
                            gcloud run services add-iam-policy-binding ${IMAGE_NAME} \
                                --region us-central1 \
                                --member='allUsers' \
                                --role='roles/run.invoker'
                        """
                    }
                }
            }
        }

        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }
    }
}
