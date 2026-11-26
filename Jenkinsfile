
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'dockernavaneeth/mediplus'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/Navaneethkrishna-coder/mediplus.git'
            }
        }

        stage('Django Application Check') {
            steps {
                sh '''
                    docker build -t mediplus-test:${BUILD_NUMBER} .
                    docker run --rm \
                        -e DJANGO_SECRET_KEY='temporary-ci-check-key' \
                        -e DJANGO_ALLOWED_HOSTS='*' \
                        mediplus-test:${BUILD_NUMBER} \
                        python manage.py check
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
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
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-credentials',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )
                ]) {
                    sh '''
                        git config user.name "Jenkins CI"
                        git config user.email "jenkins@example.com"

                        sed -i "s|image: .*|image: ${DOCKER_IMAGE}:${BUILD_NUMBER}|" k8s/deployment.yaml

                        git add k8s/deployment.yaml
                        git commit -m "Update MediPlus image to build ${BUILD_NUMBER}" || true

                        git -c http.extraheader="AUTHORIZATION: basic $(printf '%s:%s' "$GIT_USER" "$GIT_TOKEN" | base64 | tr -d '\\n')" \
                            push origin HEAD:main
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

