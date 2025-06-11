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
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Déterminer le namespace basé sur la branche
                    def namespace = "${IMAGE_TAG}"
                    def nodePortMovie = ""
                    def nodePortCast = ""
                    
                    // Définir les ports par environnement
                    switch(namespace) {
                        case 'dev':
                            nodePortMovie = "30001"
                            nodePortCast = "30002"
                            break
                        case 'qa':
                            nodePortMovie = "30003"
                            nodePortCast = "30004"
                            break
                        case 'staging':
                            nodePortMovie = "30005"
                            nodePortCast = "30006"
                            break
                        case 'main':
                            namespace = 'prod'
                            nodePortMovie = "30007"
                            nodePortCast = "30008"
                            break
                    }
                    
                    sh """
                        echo "Deploying to namespace: ${namespace}"
                        echo "Movie Service NodePort: ${nodePortMovie}"
                        echo "Cast Service NodePort: ${nodePortCast}"
                        
                        # Deploy Movie Service
                        helm upgrade --install movie-service-${namespace} ./charts \\
                            --namespace ${namespace} \\
                            --create-namespace \\
                            --set image.repository=amiraabidi/movie-service \\
                            --set image.tag=${IMAGE_TAG} \\
                            --set service.nodePort=${nodePortMovie} \\
                            --set fullnameOverride=movie-service-${namespace}
                        
                        # Deploy Cast Service  
                        helm upgrade --install cast-service-${namespace} ./charts \\
                            --namespace ${namespace} \\
                            --create-namespace \\
                            --set image.repository=amiraabidi/cast-service \\
                            --set image.tag=${IMAGE_TAG} \\
                            --set service.nodePort=${nodePortCast} \\
                            --set fullnameOverride=cast-service-${namespace}
                        
                        echo "Deployment completed successfully!"
                        echo "Checking deployments in namespace ${namespace}:"
                        kubectl get pods -n ${namespace}
                        kubectl get services -n ${namespace}
                    """
                }
            }
        }
    }
}
