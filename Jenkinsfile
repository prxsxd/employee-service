pipeline {
    agent any

    environment {
        IMAGE_NAME = "prasad5657/employee-service"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {

                sh '''
                sed -i "s|IMAGE_PLACEHOLDER|${IMAGE_NAME}:${IMAGE_TAG}|g" k8s/deployment.yaml
                '''

                sh 'kubectl apply -f k8s/'
            }
        }
    }

    post {

        success {
            echo 'Application deployed successfully.'
        }

        failure {

            script {

                echo 'Pipeline failed.'

                def logs = currentBuild.rawBuild.getLog(100).join('\n')

                writeFile file: 'failure-log.txt', text: logs
            }

            sh '''
            curl -X POST http://host.docker.internal:1001/api/analyze \
            -H "Content-Type: text/plain" \
            --data-binary @failure-log.txt
            '''
        }
    }
}