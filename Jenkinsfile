pipeline { 
    agent any 
 
    environment { 
        PROJECT_ID = 'modern-spirit-475318-r3' 
        GCP_CREDENTIALS = credentials('gcp-keyfile') // Add this in Jenkins credentials 
        FRONTEND_IMAGE = 'gcr.io/modern-spirit-475318-r3/frontend'
        BACKEND_IMAGE = 'gcr.io/modern-spirit-475318-r3/backend'
    } 
 
    stages { 
        stage('Checkout') { 
            steps { 
                dir('frontend') {
                    git branch: 'main', url: 'https://github.com/sleepycoder-dsh/FrondEndDevelopment'
                }
                dir('backend') { 
                    git branch: 'main', url: 'https://github.com/sleepycoder-dsh/BackendDevelopment' 
                } 
            } 
        } 
 
        stage('Build Frontend Docker Image') { 
            steps { 
                dir('frontend') { 
                    sh 'docker build -t $FRONTEND_IMAGE:latest .' 
                } 
            } 
        } 
 
        stage('Build Backend Docker Image') { 
            steps { 
                dir('backend') { 
                    sh './mvnw clean package -DskipTests' 
                    sh 'docker build -t $BACKEND_IMAGE:latest .' 
                } 
            } 
        } 
 
        stage('Push to GCR') { 
            steps { 
                withCredentials([file(credentialsId: 'gcp-keyfile', variable: 
'GOOGLE_APPLICATION_CREDENTIALS')]) { 
                    sh ''' 
                    gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS 
                    gcloud auth configure-docker 
                    docker push $FRONTEND_IMAGE:latest 
                    docker push $BACKEND_IMAGE:latest 
                    ''' 
                } 
            } 
        } 
 
        stage('Deploy to GKE') { 
            steps { 
                withCredentials([file(credentialsId: 'gcp-keyfile', variable: 
'GOOGLE_APPLICATION_CREDENTIALS')]) { 
                    sh ''' 
                    gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS 
                    gcloud container clusters get-credentials webapp-cluster --zone us-central1-c --project $PROJECT_ID 
                    kubectl apply -f k8s/backend-deployment.yaml 
                    kubectl apply -f k8s/backend-service.yaml 
                    kubectl apply -f k8s/frontend-deployment.yaml 
                    kubectl apply -f k8s/frontend-service.yaml 
                    kubectl apply -f k8s/ingress.yaml 
                    ''' 
                } 
            } 
        } 
    } 
}
