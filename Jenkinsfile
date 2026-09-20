pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '5'))   // history stays clean forever
    }

    environment {
        // ⚠️ match these two IDs to YOUR Jenkins credentials (Manage Jenkins → Credentials)
        DOCKER_CREDS = 'docker-hub-credentials'
        KUBE_CREDS   = 'kubeconfig-file-credentials'
        REGISTRY     = 'mhyderali004'
        BACKEND_IMG  = "${REGISTRY}/cicd-backend"
        FRONTEND_IMG = "${REGISTRY}/cicd-frontend"
        TAG          = 'latest'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn -f backend/pom.xml clean package -q'   // adjust path if your pom lives elsewhere
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDS,
                                                  usernameVariable: 'DH_USER',
                                                  passwordVariable: 'DH_PASS')]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker build -t ${BACKEND_IMG}:${TAG}  ./backend
                        docker build -t ${FRONTEND_IMG}:${TAG} ./frontend
                        docker push ${BACKEND_IMG}:${TAG}
                        docker push ${FRONTEND_IMG}:${TAG}
                        docker logout
                    '''
                }
            }
        }

        stage('Validate Manifests') {          // NEW: catches YAML/schema errors BEFORE touching the cluster
            steps {
                withCredentials([file(credentialsId: env.KUBE_CREDS, variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply --dry-run=client -f k8s/'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: env.KUBE_CREDS, variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl apply -f k8s/
                        kubectl rollout restart deployment/backend deployment/frontend deployment/mysql
                    '''
                }
            }
        }

        stage('Verify Rollout') {             // the stage that stops green builds from lying
            steps {
                withCredentials([file(credentialsId: env.KUBE_CREDS, variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl rollout status deployment/mysql    --timeout=180s
                        kubectl rollout status deployment/backend  --timeout=180s
                        kubectl rollout status deployment/frontend --timeout=180s
                        kubectl get pods -n default
                    '''
                }
            }
        }
    }

    post {
        success { echo '✅ Pipeline green: images built, manifests valid, rollout verified.' }
        failure { echo '🚨 Pipeline failed — inspect the red stage; cluster keeps last known-good state.' }
        always  { cleanWs() }                  // fresh workspace every build = no stale-file surprises
    }
}
