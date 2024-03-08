pipeline {
    agent any
    environment {
        dockerHubRegistry = 'yoisakikanade/fligh7'
        dockerHubRegistryCredential = 'yoisakikanade0'
        githubCredential = 'yoisakikanade'
        PROJECT_ID = 'fligh7'
        GCP_ZONE_1 = 'asia-northeast3'
        GCP_ZONE_2 = 'asia-northeast1'
        CLUSTER_NAME_1 = 'my-cluster-seoul-1'
        CLUSTER_NAME_2 = 'my-cluster-tokyo-1'
        ARTIFACT_REPO = 'asia-northeast3-docker.pkg.dev/fligh7/fligh7-image'
    }
  
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
            post {
                failure {
                    echo 'repository checkout failure'
                }
                success {
                    echo 'repository checkout success'
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                sh "docker build . -t ${dockerHubRegistry}:${currentBuild.number}"
                sh "docker build . -t ${dockerHubRegistry}:latest"
            }

            post {
                failure {
                    echo 'Docker image build failure !'
                }
                success {
                    echo 'Docker image build success !'
                }
            }
        }

        stage('Docker Image Push') {
            steps {
                withDockerRegistry([ credentialsId: dockerHubRegistryCredential, url: ""]) {
                    sh "docker push ${dockerHubRegistry}:${BUILD_NUMBER}"
                    sh "docker push ${dockerHubRegistry}:latest"
                }
            }
            post {
                success {
                    echo 'Docker image push success!'
                }
                failure {
                    echo 'Docker image push failure!'
                }
            }
        }   

    
        stages {
            stage('Build and Push Docker Image') {
                steps {
                    script {
                        def dockerImageTag = "asia-northeast3-docker.pkg.dev/fligh7/fligh7-image/yoisakikanade/fligh7:${BUILD_NUMBER}"
                        docker.build(dockerImageTag, "-f Dockerfile .")
                        docker.withRegistry('https://asia-northeast3-docker.pkg.dev', 'yoisakikanade1') {
                            docker.image(dockerImageTag).push()
                        }
                    }
                }
            }
        }
    
        post {
            success {
                echo 'Update Artifact Registry success!'
            }
            failure {
                echo 'Update Artifact Registry failure!'
            }
        }
        
        stage('Deploy to GKE 1') {
            steps {
                sh "gcloud container clusters get-credentials $CLUSTER_NAME_1 --zone $GCP_ZONE_1 --project $PROJECT_ID"
                sh "kubectl set image deployment/$DEPLOYMENT_NAME $CONTAINER_NAME=$dockerHubRegistry:$BUILD_NUMBER"
            }
        }
        stage('Deploy to GKE 2') {
            steps {
                sh "gcloud container clusters get-credentials $CLUSTER_NAME_2 --zone $GCP_ZONE_2 --project $PROJECT_ID"
                sh "kubectl set image deployment/$DEPLOYMENT_NAME $CONTAINER_NAME=$dockerHubRegistry:$BUILD_NUMBER"
            }
        }
    }
    post {
        always {
            echo 'Deployment completed'
        }
    }
}
