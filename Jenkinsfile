pipeline {
    agent { label 'slave' }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('546603df-2872-47f3-97df-1843a8662bde')
    }

    stages {
        stage('SCM Checkout') {
            steps {
                echo 'Cloning the repo'
                git 'https://github.com/abm2707/star-agile-insurance-project.git'
            }
        }
        stage('Application Build') {
            steps {
                echo 'Building the application'
                sh 'mvn clean package'
            }
        }
        stage('Docker Build') {
            steps {
                echo 'Building Docker image'
                sh "docker build -t akhilbmenon/staragile-insureme:${BUILD_NUMBER} ."
                sh "docker tag akhilbmenon/staragile-insureme:${BUILD_NUMBER} akhilbmenon/staragile-insureme:latest"
            }
        }
        stage('Docker Push') {
            steps {
                echo 'Pushing Docker image to DockerHub'
                sh 'echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "$DOCKERHUB_CREDENTIALS_USR" --password-stdin'
                sh "docker push akhilbmenon/staragile-insureme:latest"
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'KubernetesMaster',
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'kdeploy.yaml',
                                        remoteDirectory: '.',
                                        execCommand: 'kubectl apply -f kdeploy.yaml',
                                        execTimeout: 120000
                                    )
                                ],
                                verbose: true
                            )
                        ]
                    )
                }
            }
        }
    }
}
