pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = "tripmate-ai"
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                sh 'ls -la'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Syntax Check') {
            steps {
                // Compile-check the code inside the freshly built image.
                // No env vars / DB needed, so no secrets are involved in CI.
                sh "docker run --rm ${IMAGE_NAME}:${IMAGE_TAG} python -m py_compile app.py backend.py mcp_client.py"
            }
        }

        stage('Deploy to Render') {
            when {
                expression { env.GIT_BRANCH == 'origin/main' || env.GIT_BRANCH == 'main' }
            }
            steps {
                withCredentials([string(credentialsId: 'render-deploy-hook', variable: 'RENDER_DEPLOY_HOOK')]) {
                    sh 'curl -fsS -X POST "$RENDER_DEPLOY_HOOK"'
                }
            }
        }
    }

    post {
        success { echo "Pipeline finished: image built, checked, deploy triggered." }
        failure { echo "Build failed - check the stage logs above." }
        always  { sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true" }
    }
}
