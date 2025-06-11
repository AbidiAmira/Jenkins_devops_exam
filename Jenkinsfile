pipeline {
    agent any
    
    environment {
        DOCKERHUB_USERNAME = "amiraabidi"
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials"
        IMAGE_TAG = "${env.BRANCH_NAME}"
        MOVIE_IMAGE = "${DOCKERHUB_USERNAME}/movie_service:${IMAGE_TAG}"
    }
    
    stages {
        stage('Build Movie Service') {
            steps {
                script {
                    sh """
                        echo "Building Movie Service image: ${MOVIE_IMAGE}"
                        docker build -t ${MOVIE_IMAGE} ./movie_service/
                        echo "Movie Service build completed successfully"
                        docker images | grep movie_service
                    """
                }
            }
        }
    }
}
