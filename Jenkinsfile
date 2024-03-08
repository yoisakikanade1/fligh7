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
        GOOGLE_APPLICATION_CREDENTIALS = credentials('fligh7-a8c1b4c708f2.json')
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

        stage('Build and Push Docker Image to Docker Hub') {
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

        stage('Docker Image Push to Docker Hub') {
            steps {
                withDockerRegistry([ credentialsId: dockerHubRegistryCredential, url: ""]) {
                    sh "docker push ${dockerHubRegistry}:${BUILD_NUMBER}"
                    sh "docker push ${dockerHubRegistry}:latest"
                }
            }
            post {
                success {
                    echo 'Docker image push to Docker Hub success!'
                }
                failure {
                    echo 'Docker image push to Docker Hub failure!'
                }
            }
        }   

        stage('Tag and Push to Artifact Registry') {
            steps {
                script {
                    // 도커 허브에 푸시한 이미지를 Artifact Registry에 태깅하고 푸시
                    def dockerImageTag = "asia-northeast3-docker.pkg.dev/${ARTIFACT_REPO}/yoisakikanade/fligh7:${BUILD_NUMBER}"
                    sh "docker tag ${dockerHubRegistry}:${BUILD_NUMBER} ${dockerImageTag}"
                    withCredentials([file(credentialsId: 'GOOGLE_APPLICATION_CREDENTIALS', variable: 'GOOGLE_APPLICATION_CREDENTIALS_JSON')]) {
                        sh "DOCKER_CONFIG=${env.HOME}/.docker/google-config docker --config ${env.HOME}/.docker push ${dockerImageTag}"
                    }
                }
            }
            post {
                success {
                    echo 'Tag and push to Artifact Registry success!'
                }
                failure {
                    echo 'Tag and push to Artifact Registry failure!'
                }
            }
        }

        stage('Deploy to GKE 1') {
            steps {
                sh "gcloud container clusters get-credentials $CLUSTER_NAME_1 --zone $GCP_ZONE_1 --project $PROJECT_ID"
                sh "kubectl set image deployment/$DEPLOYMENT_NAME $CONTAINER_NAME=${dockerHubRegistry}:${BUILD_NUMBER}"
            }
        }
        stage('Deploy to GKE 2') {
            steps {
                sh "gcloud container clusters get-credentials $CLUSTER_NAME_2 --zone $GCP_ZONE_2 --project $PROJECT_ID"
                sh "kubectl set image deployment/$DEPLOYMENT_NAME $CONTAINER_NAME=${dockerHubRegistry}:${BUILD_NUMBER}"
            }
        }
    }
    post {
        always {
            echo 'Deployment completed'
        }
    }
}
