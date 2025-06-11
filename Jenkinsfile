pipeline {
    agent any
    
    environment {
        DOCKERHUB_USERNAME = "amiraabidi"
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials"
        IMAGE_TAG = "${env.GIT_BRANCH?.tokenize('/')[-1] ?: 'latest'}"
        MOVIE_IMAGE = "${DOCKERHUB_USERNAME}/movie-service:${IMAGE_TAG}"
        CAST_IMAGE = "${DOCKERHUB_USERNAME}/cast-service:${IMAGE_TAG}"
    }
    
    stages {        
        stage('Build Movie Service') {
            steps {
                script {
                    sh """
                        echo "Building Movie Service image: ${MOVIE_IMAGE}"
                        docker build -t ${MOVIE_IMAGE} ./movie-service/
                        echo "Movie Service build completed successfully"
                        docker images | grep movie-service
                    """
                }
            }
        }
        
        stage('Build Cast Service') {
            steps {
                script {
                    sh """
                        echo "Building Cast Service image: ${CAST_IMAGE}"
                        docker build -t ${CAST_IMAGE} ./cast-service/
                        echo "Cast Service build completed successfully"
                        docker images | grep cast-service
                    """
                }
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", 
                                                    usernameVariable: 'DOCKER_USERNAME', 
                                                    passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                            echo "Logging into DockerHub..."
                            echo \${DOCKER_PASSWORD} | docker login -u \${DOCKER_USERNAME} --password-stdin
                            
                            echo "Pushing Movie Service image: ${MOVIE_IMAGE}"
                            docker push ${MOVIE_IMAGE}
                            
                            echo "Pushing Cast Service image: ${CAST_IMAGE}"
                            docker push ${CAST_IMAGE}
                            
                            echo "Both images pushed successfully to DockerHub!"
                            docker logout
                        """
                    }
                }
            }
        }
    }
}
