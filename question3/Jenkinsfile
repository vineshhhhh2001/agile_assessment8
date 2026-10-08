pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = "vineshhhhh2001"
        IMAGE_NAME_PROFILE = "profile-app"
        IMAGE_NAME_EVENT   = "event-app"
        IMAGE_NAME_DEPT    = "dept-portal"
        IMAGE_TAG          = "latest"
        DOCKERHUB_CREDS    = credentials("dockerhub")
    }

    stages {

        // ─────────────────────────────────────────────────────────────────
        // STAGE 1 – Clone code from GitHub
        // ─────────────────────────────────────────────────────────────────
        stage("Clone Code") {
            steps {
                echo "Cloning repository..."
                git branch: "main",
                    url: "https://github.com/vineshhhhh2001/agile_assessment8.git"
            }
        }

        // ─────────────────────────────────────────────────────────────────
        // STAGE 2 – Build Docker images for all three applications
        // ─────────────────────────────────────────────────────────────────
        stage("Build Docker Images") {
            parallel {
                stage("Build Profile App (Q1)") {
                    steps {
                        echo "Building profile-app image..."
                        bat """
                            copy /Y index.html index_build.html
                            docker build -f Dockerfile -t %DOCKER_HUB_USER%/%IMAGE_NAME_PROFILE%:%IMAGE_TAG% .
                            del index_build.html
                        """
                    }
                }
                stage("Build Event App (Q2)") {
                    steps {
                        echo "Building event-app image..."
                        bat """
                            docker build -f Dockerfile.event -t %DOCKER_HUB_USER%/%IMAGE_NAME_EVENT%:%IMAGE_TAG% .
                        """
                    }
                }
                stage("Build Dept Portal (Q3)") {
                    steps {
                        echo "Building dept-portal image..."
                        bat """
                            docker build -f Dockerfile.dept -t %DOCKER_HUB_USER%/%IMAGE_NAME_DEPT%:%IMAGE_TAG% .
                        """
                    }
                }
            }
        }

        // ─────────────────────────────────────────────────────────────────
        // STAGE 3 – Push all images to Docker Hub
        // ─────────────────────────────────────────────────────────────────
        stage("Push Images to Docker Hub") {
            steps {
                echo "Logging in to Docker Hub..."
                bat """
                    echo %DOCKERHUB_CREDS_PSW% | docker login -u %DOCKERHUB_CREDS_USR% --password-stdin
                    docker push %DOCKER_HUB_USER%/%IMAGE_NAME_PROFILE%:%IMAGE_TAG%
                    docker push %DOCKER_HUB_USER%/%IMAGE_NAME_EVENT%:%IMAGE_TAG%
                    docker push %DOCKER_HUB_USER%/%IMAGE_NAME_DEPT%:%IMAGE_TAG%
                    docker logout
                """
            }
        }

        // ─────────────────────────────────────────────────────────────────
        // STAGE 4 – Deploy to Kubernetes (Docker Desktop)
        // ─────────────────────────────────────────────────────────────────
        stage("Deploy to Kubernetes") {
            steps {
                echo "Applying Kubernetes manifests..."
                bat """
                    kubectl apply -f deployment.yaml
                    kubectl rollout restart deployment/profile-app
                    kubectl rollout restart deployment/event-app
                    kubectl rollout restart deployment/dept-portal
                """
            }
        }

        // ─────────────────────────────────────────────────────────────────
        // STAGE 5 – Verify deployments
        // ─────────────────────────────────────────────────────────────────
        stage("Verify Deployment") {
            steps {
                echo "Checking pod status..."
                bat """
                    kubectl get pods -o wide
                    kubectl get services
                    kubectl rollout status deployment/profile-app --timeout=120s
                    kubectl rollout status deployment/event-app   --timeout=120s
                    kubectl rollout status deployment/dept-portal --timeout=120s
                """
            }
        }
    }

    post {
        success {
            echo "=============================="
            echo " Pipeline SUCCESS!"
            echo " Profile App  -> http://localhost:30010"
            echo " Event App    -> http://localhost:30011"
            echo " Dept Portal  -> http://localhost:30012"
            echo "=============================="
        }
        failure {
            echo "Pipeline FAILED – check the logs above."
        }
        always {
            cleanWs()
        }
    }
}
