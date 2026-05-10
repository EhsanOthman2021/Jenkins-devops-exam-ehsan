pipeline {
    agent any

    environment {
        DOCKER_ID = "ehsanothman2025"
        TAG = "v.${BUILD_ID}"
    }

    stages {

        stage('Build Images') {
            steps {
                sh '''
                docker build -t $DOCKER_ID/movie-service:$TAG ./movie-service
                docker build -t $DOCKER_ID/cast-service:$TAG ./cast-service
                '''
            }
        }

        stage('Push Images') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }

            steps {
                sh '''
                echo $DOCKER_PASS | docker login -u $DOCKER_ID --password-stdin

                docker push $DOCKER_ID/movie-service:$TAG
                docker push $DOCKER_ID/cast-service:$TAG
                '''
            }
        }

        stage('Deploy Dev') {
            environment {
                KUBECONFIG = credentials("config")
            }

            steps {
                sh '''
                mkdir -p .kube
                cat $KUBECONFIG > .kube/config

                helm upgrade --install app-dev ./charts \
                --namespace dev \
                --create-namespace
                '''
            }
        }

        stage('Deploy QA') {
            environment {
                KUBECONFIG = credentials("config")
            }

            steps {
                sh '''
                mkdir -p .kube
                cat $KUBECONFIG > .kube/config

                helm upgrade --install app-qa ./charts \
                --namespace qa \
                --create-namespace
                '''
            }
        }

        stage('Deploy Staging') {
            environment {
                KUBECONFIG = credentials("config")
            }

            steps {
                sh '''
                mkdir -p .kube
                cat $KUBECONFIG > .kube/config

                helm upgrade --install app-staging ./charts \
                --namespace staging \
                --create-namespace
                '''
            }
        }

        stage('Deploy Prod') {

            when {
                expression {
                    return env.BRANCH_NAME == 'master' || env.GIT_BRANCH == 'origin/master'
                }
            }

            environment {
                KUBECONFIG = credentials("config")
            }

            steps {

                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Deploy to production?', ok: 'Yes'
                }

                sh '''
                mkdir -p .kube
                cat $KUBECONFIG > .kube/config

                helm upgrade --install app-prod ./charts \
                --namespace prod \
                --create-namespace
                '''
            }
        }
    }

    post {

        success {
            echo "Pipeline completed successfully"
        }

        failure {
            mail to: "YOUR_EMAIL@gmail.com",
                 subject: "${env.JOB_NAME} - Build #${env.BUILD_ID} FAILED",
                 body: """
Pipeline failed.

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_ID}

Check console output here:
${env.BUILD_URL}
"""
        }
    }
}