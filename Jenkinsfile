pipeline {
    agent any
    
    environment {
        DOCKERHUB_USERNAME = "amiraabidi"
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials"
        IMAGE_TAG = "${env.GIT_BRANCH?.tokenize('/')[-1] ?: 'latest'}"
        MOVIE_IMAGE = "${DOCKERHUB_USERNAME}/movie_service:${IMAGE_TAG}"
    }
    
    stages {
        stage('Debug - Check Structure') {
            steps {
                script {
                    sh """
                        echo "=== DEBUG INFORMATION ==="
                        echo "Current branch: ${env.BRANCH_NAME}"
                        echo "Current directory: \$(pwd)"
                        echo "Files and folders:"
                        ls -la
                        echo "=== Looking for Dockerfiles ==="
                        find . -name "Dockerfile" -type f
                        echo "=== Complete directory structure ==="
                        find . -type d
                    """
                }
            }
        }
        
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
