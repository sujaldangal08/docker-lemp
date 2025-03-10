pipeline {
    agent any

    environment {
        IMAGE_NAME = 'sujaldangal/nginx-app' // Image name for your Nginx app
    }

    stages {
        stage('Checkout Repository') {
            steps {
                // Checkout your repository containing the docker-compose.yml and other files
                git url: 'https://github.com/sujaldangal08/docker-lemp.git', branch: 'main'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                usernameVariable: 'DOCKER_USER', 
                                passwordVariable: 'DOCKER_PASS')]) {
                    // Login to Docker Hub
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Run Docker Compose to Build and Start Containers') {
            steps {
                // Build and run the containers defined in docker-compose.yml
                sh 'docker compose up -d --build'
            }
        }

        stage('Extract Image ID from Docker Compose') {
            steps {
                script {
                    // Extract image name from the docker-compose build
                    def imageID = sh(script: "docker images --format '{{.Repository}}:{{.Tag}}' | grep '$IMAGE_NAME'", returnStdout: true).trim()
                    if (!imageID) {
                        error "No image found with name $IMAGE_NAME"
                    }
                    env.NEW_IMAGE_TAG = imageID
                }
            }
        }

        stage('Push New Image to Docker Hub') {
            steps {
                script {
                    // Tag the image and push it to Docker Hub with version tag and latest tag
                    def buildTag = "v${env.BUILD_NUMBER}"
                    sh "docker tag $NEW_IMAGE_TAG $IMAGE_NAME:${buildTag}"
                    sh "docker push $IMAGE_NAME:${buildTag}"
                    sh "docker push $IMAGE_NAME:latest"
                }
            }
        }
    }
}

