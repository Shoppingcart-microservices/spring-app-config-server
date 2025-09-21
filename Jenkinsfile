pipeline {

    environment {
        PROJECT_ID        = "${PROJECT_ID}"
        REGISTRY_URL      = "${REGISTRY_URL}"
        ARTIFACT_REGISTRY = "${ARTIFACT_REGISTRY}"
        IMAGE_NAME        = "config-server"
        CLUSTER_NAME      = "${CLUSTER}"
        LOCATION          = "${ZONE}"
        REPO_URL          = "${REGISTRY_URL}/${PROJECT_ID}/${ARTIFACT_REGISTRY}"
    }

    agent any

    stage("Checkout Git Branch") {
        steps {
            git branch: 'develop'
            url: 'https://github.com/Shoppingcart-microservices/spring-app-config-server.git'
            credentialsId: 'blabla'
        }
    }
    stage("Build and Push Image") {
        steps {
            withCredentials([file(credentialsId: 'blabla', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                sh """
                        echo "Activating GCP service account..."
                        gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                        gcloud config set project ${PROJECT_ID}

                        echo "Configuring Docker authentication..."
                        gcloud auth configure-docker ${REGISTRY_URL} --quiet

                        echo "Building and pushing image with Jib..."
                        ${mvnCMD} clean install jib:build -DREPO_URL=${repourl}
                    """
            }
        }
    }
    stage("Deploy to GKE (Google k8s Engine)") {
        sh "sed -i 's|IMAGE_URL|${repourl}|g' k8s/config-server-deployment.yaml"
        step([
                $class: 'KubernetesEngineBuilder',
                projectId: env.PROJECT_ID,
                clusterName: env.CLUSTER_NAME,
                location: env.LOCATION,
                manifestPattern: 'k8s/config-server-deployment.yaml',
                credentialsId: 'blabla',
                verifyDeployments: true])
    }
}