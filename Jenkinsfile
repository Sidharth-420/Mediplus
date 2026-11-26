
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'sidharth18/mediplus'
    }

    stages {

        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Django Application Check') {
            steps {
                sh '''
                    docker build -t mediplus-test:${BUILD_NUMBER} .

                    docker run --rm \
                        -e DJANGO_SECRET_KEY='temporary-ci-check-key' \
                        -e DJANGO_ALLOWED_HOSTS='localhost,127.0.0.1' \
                        mediplus-test:${BUILD_NUMBER} \
                        python manage.py check
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                        -t ${DOCKER_IMAGE}:latest .
                '''
            }
        }

        stage('Docker Hub Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'MediPlus CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'MediPlus CI/CD pipeline failed. Check the console output.'
        }
    }
}
