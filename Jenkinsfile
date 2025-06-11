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
                        
                        # Create namespace if it doesn't exist
                        kubectl create namespace ${namespace} --dry-run=client -o yaml | kubectl apply -f -
                        
                        # Create Movie Service Deployment
                        cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: movie-service-${namespace}
  namespace: ${namespace}
spec:
  replicas: 2
  selector:
    matchLabels:
      app: movie-service-${namespace}
  template:
    metadata:
      labels:
        app: movie-service-${namespace}
    spec:
      containers:
      - name: movie-service
        image: ${MOVIE_IMAGE}
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URI
          value: "sqlite:///./movies.db"
        - name: DATABASE_URL
          value: "sqlite:///./movies.db"
        readinessProbe:
          httpGet:
            path: /api/v1/checkapi
            port: 8000
          initialDelaySeconds: 15
          periodSeconds: 10
EOF

                        # Create Movie Service Service
                        cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: movie-service-${namespace}
  namespace: ${namespace}
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8000
    nodePort: ${nodePortMovie}
  selector:
    app: movie-service-${namespace}
EOF

                        # Create Cast Service Deployment
                        cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cast-service-${namespace}
  namespace: ${namespace}
spec:
  replicas: 2
  selector:
    matchLabels:
      app: cast-service-${namespace}
  template:
    metadata:
      labels:
        app: cast-service-${namespace}
    spec:
      containers:
      - name: cast-service
        image: ${CAST_IMAGE}
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URI
          value: "sqlite:///./casts.db"
        - name: DATABASE_URL
          value: "sqlite:///./casts.db"
        readinessProbe:
          httpGet:
            path: /api/v1/checkapi
            port: 8000
          initialDelaySeconds: 15
          periodSeconds: 10
EOF

                        # Create Cast Service Service
                        cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: cast-service-${namespace}
  namespace: ${namespace}
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8000
    nodePort: ${nodePortCast}
  selector:
    app: cast-service-${namespace}
EOF

                        echo "Deployment completed successfully!"
                        echo "Checking deployments in namespace ${namespace}:"
                        kubectl get pods -n ${namespace}
                        kubectl get services -n ${namespace}
                        echo ""
                        echo "Services will be available at:"
                        echo "Movie Service: http://your-k8s-node:${nodePortMovie}/api/v1/movies/docs"
                        echo "Cast Service: http://your-k8s-node:${nodePortCast}/api/v1/casts/docs"
                    """
                }
            }
        }
    }
}
