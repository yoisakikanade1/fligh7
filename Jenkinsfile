pipeline {
    agent any
    environment {
        dockerHubRegistry = 'yoisakikanade/fligh7'
        dockerHubRegistryCredential = 'yoisakikanade0'
        githubCredential = 'yoisakikanade'
        gcpCredential = 'yoisakikanade1'
        PROJECT_ID = 'fligh7'
        GCP_ZONE_1 = 'asia-northeast3'
        GCP_ZONE_2 = 'asia-northeast1'
        CLUSTER_NAME_1 = 'my-cluster-seoul-1'
        CLUSTER_NAME_2 = 'my-cluster-tokyo-1'
        ARTIFACT_REPO = 'asia-northeast3-docker.pkg.dev/fligh7/fligh7-image'
        GOOGLE_APPLICATION_CREDENTIALS = credentials('yoisakikanade1')
        GCLOUD_PATH = '/usr/bin/gcloud'
        GCP_CREDENTIAL_FILE = '/home/consecrator/fligh7-jenkins-key.json'
        kubernetesNamespace = "default"  // 배포할 Deployment가 있는 네임스페이스를 지정합니다.
        deploymentjobName = "kubernetes-manifests-deployer-job"  // 새 배포를 위한 job의 이름을 지정합니다.
        dockerImageTag = "${env.ARTIFACT_REPO}/yoisakikanade/fligh7:${currentBuild.number}"  // Artifact Registry에 업로드된 이미지의 태그를 지정합니다.
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
                    sh "docker push ${dockerHubRegistry}:${currentBuild.number}"
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
                    def dockerImageTag = "${env.ARTIFACT_REPO}/yoisakikanade/fligh7:${currentBuild.number}"
                    sh "docker tag ${env.dockerHubRegistry}:${currentBuild.number} ${dockerImageTag}"
                    sh "sudo gcloud auth configure-docker asia-northeast3-docker.pkg.dev"
                    sh "docker push ${dockerImageTag}"
                    sh "docker rmi ${env.dockerHubRegistry}:${currentBuild.number}"  // 여기 수정
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
        
        stage('Deploy to GKE') {
            steps {
                script {
                    // 현재 배포된 Deployment의 이미지를 업데이트합니다.
                    sh "kubectl set image job/${deploymentjobName} *=${dockerImageTag} --namespace=${kubernetesNamespace}"
                }
            }
            post {
                success {
                    echo 'Deployment to GKE success!'
                }
                failure {
                    echo 'Deployment to GKE failure!'
                }
            }
        }



    }
    post {
        always {
            echo 'Deployment completed'
        }
    }
}
