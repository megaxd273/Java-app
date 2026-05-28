pipeline {
    agent {
        kubernetes {
            label 'kaniko'
            defaultContainer 'kaniko'
        }
    }

    environment {
        HARBOR = 'harbor.local'
        BACK_IMAGE = "${HARBOR}/library/java-backend"
        FRONT_IMAGE = "${HARBOR}/library/react-frontend"
        COMMIT = "${env.GIT_COMMIT?.take(8) ?: 'latest'}"
    }

    stages {

        stage('Build Backend') {
            steps {
                container('kaniko') {
                    sh '''
                        /kaniko/executor \
                          --context=${WORKSPACE}/back-end \
                          --dockerfile=${WORKSPACE}/back-end/dockerfile \
                          --destination=${BACK_IMAGE}:${COMMIT} \
                          --destination=${BACK_IMAGE}:latest \
                          --cache=true \
                          --cache-repo=${HARBOR}/library/cache \
                          --skip-tls-verify \
                          --ignore-path=/product_uuid \
                          --ignore-path=/proc \
                          --ignore-path=/sys
                    '''
                }
            }
        }

        stage('Build Frontend') {
            steps {
                container('kaniko') {
                    sh '''
                        /kaniko/executor \
                          --context=${WORKSPACE}/front-end \
                          --dockerfile=${WORKSPACE}/front-end/dockerfile \
                          --destination=${FRONT_IMAGE}:${COMMIT} \
                          --destination=${FRONT_IMAGE}:latest \
                          --cache=true \
                          --cache-repo=${HARBOR}/library/cache \
                          --skip-tls-verify \
                          --ignore-path=/product_uuid \
                          --ignore-path=/proc \
                          --ignore-path=/sys
                    '''
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                container('helm') {
                    sh '''
                        helm upgrade --install backend ${WORKSPACE}/charts/backend \
                          --namespace app \
                          --set image.tag=${COMMIT}
                    '''
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                container('helm') {
                    sh '''
                        helm upgrade --install frontend ${WORKSPACE}/charts/frontend \
                          --namespace app \
                          --set image.tag=${COMMIT}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed. Backend: ${BACK_IMAGE}:${COMMIT}, Frontend: ${FRONT_IMAGE}:${COMMIT}"
        }
        failure {
            echo "Pipeline failed at stage: ${currentBuild.result}"
        }
    }
}
