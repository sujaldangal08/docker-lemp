pipeline {
    agent any

    environment {
        IMAGE_NAME_NGINX = 'sujaldangal/nginx-app'  // Image name for your Nginx app
    }

    stages {
        stage('Checkout Repository') {
            steps {
                // Checkout your repository containing the docker-compose.yml and other files
                git url: 'https://github.com/sujaldangal08/docker-lemp.git', branch: 'master'
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
        	    // Get the list of images after running docker compose up
        	    def images = sh(script: "docker images --format '{{.Repository}}:{{.Tag}}'", returnStdout: true).trim()

        	    // Debug output for the list of images
        	    echo "Docker images list: ${images}"

	            // Check if the desired image is in the list
        	    def nginxImageID = images.find { it.contains('sujaldangal/nginx-app') }

        	    if (!nginxImageID) {
        	        error "No image found for Nginx with name sujaldangal/nginx-app"
        	    }

       		    echo "Found image ID: ${nginxImageID}"
        	    env.NEW_NGINX_IMAGE_TAG = nginxImageID
 	       }
 	   }
	}
		

        stage('Push New Image to Docker Hub') {
            steps {
                script {
                    // Tag the Nginx image and push it to Docker Hub with version tag and latest tag
                    def nginxBuildTag = "v${env.BUILD_NUMBER}"
                    sh "docker tag $NEW_NGINX_IMAGE_TAG $IMAGE_NAME_NGINX:${nginxBuildTag}"
                    sh "docker push $IMAGE_NAME_NGINX:${nginxBuildTag}"
                    sh "docker push $IMAGE_NAME_NGINX:latest"
                }
            }
        }
    }

    post {
        always {
            // Cleanup Docker system to free resources
            sh 'docker system prune -f'
        }
    }
}

