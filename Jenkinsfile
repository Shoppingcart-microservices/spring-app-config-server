pipeline {

    environment {
        PROJECT_ID = "${PROJECT_ID}"
        REGISTRY_URL = "${REGISTRY_URL}"
        ARTIFACT_REGISTRY = "${ARTIFACT_REGISTRY}"
        IMAGE_NAME = "config-server"
        CLUSTER_NAME = "${CLUSTER}"
        LOCATION = "${ZONE}"
        REPO_URL = "${REGISTRY_URL}/${PROJECT_ID}/${ARTIFACT_REGISTRY}"
    }

    agent any

    stages {
        stage("Checkout Git Branch") {
            steps {
                git([
                        url          : 'https://github.com/Shoppingcart-microservices/spring-app-config-server.git',
                        branch       : 'develop',
                        credentialsId: 'git'
                ])
            }
        }
        stage("Build and Push Image") {
            steps {
                script {
                    def mvnHome = tool name: 'maven', type: 'maven'
                    def mvnCMD = "${mvnHome}/bin/mvn"

                    withCredentials([file(credentialsId: 'GCP', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh """
                        echo "Activating GCP service account..."
                        gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                        gcloud config set project ${PROJECT_ID}

                        echo "Configuring Docker authentication..."
                        gcloud auth configure-docker ${REGISTRY_URL} --quiet

                        echo "Building and pushing image with Jib..."
                        ${mvnCMD} clean install jib:build -Dimage=${REPO_URL}/${IMAGE_NAME}:latest
                    """
                    }
                }
            }
        }
        stage("Deploy to GKE (Google k8s Engine)") {
            steps {
                script {
                    sh "sed -i 's|IMAGE_URL|${REPO_URL}|g' k8s/config-server-deployment.yaml"
                }
                step([
                        $class           : 'KubernetesEngineBuilder',
                        projectId        : env.PROJECT_ID,
                        clusterName      : env.CLUSTER_NAME,
                        location         : env.LOCATION,
                        manifestPattern  : 'k8s/config-server-deployment.yaml',
                        credentialsId    : 'GCPproject',
                        verifyDeployments: true])
            }
        }
    }
}