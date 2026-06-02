pipeline {
    agent any

    environment {
        IMAGE_NAME = "ci-cd-pipeline2"
    }

    stages {
        stage("Code Cloning") {
            steps {
                echo "Cloning the code from github"
                git branch: 'main', url: 'https://github.com/Mohanm1995/DevOps-CICD-Project.git'
            }
        }

        stage("Maven Unit Test") {
            steps {
                dir("/var/lib/jenkins/workspace/Project1") {
                    sh "mvn test"
                }
            }
        }

        stage("Maven Build") {
            steps {
                dir("/var/lib/jenkins/workspace/Project1") {
                    sh "mvn clean install"
                }
            }
        }

        stage("Maven Integration Test") {
            steps {
                dir("/var/lib/jenkins/workspace/Project1") {
                    sh "mvn verify"
                }
            }
        }

        stage("Build Docker Image") {
            steps {
                echo "Building the image"
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage("Pushing the image to Dockerhub") {
            steps {
                echo "Pushing the image to DockerHub"
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]) {
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                    sh "docker tag ${IMAGE_NAME} ${env.dockerHubUser}/${IMAGE_NAME}:latest"
                    sh "docker push ${env.dockerHubUser}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                script {
                    dir("/var/lib/jenkins/workspace/Project1") {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                            sh "kubectl delete --all pods"
                            sh "kubectl apply -f deployment.yaml"
                            sh "kubectl apply -f service.yaml"
                        }
                    }
                }
            }
        }

        stage("Verify Deployment") {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                        sh "kubectl rollout status deployment/ci-cd-pipeline2 --timeout=60s"
                        sh "kubectl get pods -o wide"
                        sh "kubectl get svc"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS — Build #${env.BUILD_NUMBER} completed successfully"
            echo "Prometheus is now capturing success metrics via /prometheus endpoint"
        }
        failure {
            echo "Pipeline FAILED — Build #${env.BUILD_NUMBER} failed"
            echo "Check Grafana dashboard for failure trend"
        }
        always {
            echo "Pipeline finished — Status: ${currentBuild.currentResult}"
            echo "Duration: ${currentBuild.durationString}"
        }
    }
}

