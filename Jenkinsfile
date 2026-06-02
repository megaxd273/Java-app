pipeline {
    agent {
        kubernetes {
            label 'kaniko'
            defaultContainer 'kaniko'
        }
    }

    environment {
        HARBOR     = 'harbor.local:30443'
        BACK_IMAGE  = "${HARBOR}/library/java-backend"
        FRONT_IMAGE = "${HARBOR}/library/react-frontend"
        COMMIT      = "${env.GIT_COMMIT?.take(8) ?: 'latest'}"
    }

    stages {
        stage('Build & Deploy') {
            parallel {

                stage('Backend') {
                    when {
                        anyOf {
                            changeset "back-end/**"
                            changeset "charts/backend/**"
                            triggeredBy 'UserIdCause'
                        }
                    }
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            container('kaniko') {
                                sh '''
                                    /kaniko/executor \
                                      --context=${WORKSPACE}/back-end \
                                      --dockerfile=${WORKSPACE}/back-end/dockerfile \
                                      --destination=${BACK_IMAGE}:${COMMIT} \
                                      --destination=${BACK_IMAGE}:latest \
                                      --cache=true \
                                      --cache-repo=${HARBOR}/library/cache \
                                      --insecure-registry=harbor.local:30443 \
                                      --skip-tls-verify \
                                      --ignore-path=/product_uuid \
                                      --ignore-path=/proc \
                                      --ignore-path=/sys
                                '''
                            }
                            container('helm') {
                                sh '''
                                    helm upgrade --install backend ${WORKSPACE}/charts/backend \
                                      --namespace app \
                                      --set image.tag=${COMMIT}
                                '''
                            }
                        }
                    }
                }

                stage('Frontend') {
                    when {
                        anyOf {
                            changeset "front-end/**"
                            changeset "charts/frontend/**"
                            triggeredBy 'UserIdCause'
                        }
                    }
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            container('kaniko') {
                                sh '''
                                    /kaniko/executor \
                                      --context=${WORKSPACE}/front-end \
                                      --dockerfile=${WORKSPACE}/front-end/dockerfile \
                                      --destination=${FRONT_IMAGE}:${COMMIT} \
                                      --destination=${FRONT_IMAGE}:latest \
                                      --cache=true \
                                      --cache-repo=${HARBOR}/library/cache \
                                      --insecure-registry=harbor.local:30443 \
                                      --skip-tls-verify \
                                      --ignore-path=/product_uuid \
                                      --ignore-path=/proc \
                                      --ignore-path=/sys
                                '''
                            }
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

            }
        }
    }

    post {
        success  { echo "Deployed: ${COMMIT}" }
        unstable { echo "Partial deploy: ${COMMIT}" }
        failure  { echo "Failed: ${COMMIT}" }
    }
}