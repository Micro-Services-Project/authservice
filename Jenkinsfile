pipeline {
    agent any 
    environment {
        IMAGE_NAME = "obitomanu/authservice:${GIT_COMMIT}"
    }
    stages {
        stage ("CleanWS"){
            steps {
                cleanWs()
            }
        }
        stage("Git-Checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/Micro-Services-Project/authservice.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'Sonar'

                    withSonarQubeEnv('Sonar') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=authservice \
                            -Dsonar.projectName=authservice \
                            -Dsonar.sources=.
                        """
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'Sonar'
            }
        }
        stage("Build") {
            steps {
                sh """
                   printenv
                   docker build -t ${IMAGE_NAME} .
                   """
            }
        }
        stage("Scan") {
            steps {
                sh """ 
                   trivy image ${IMAGE_NAME} >> authservice-report.txt
                   """
                   }
        }
        stage ("push Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker') {
                        sh """
                           docker push ${IMAGE_NAME}
                           """
                    }
                }
            }
        }
    }
     post {
        always {
            sh "docker rmi ${IMAGE_NAME} || true"
            sh "docker logout || true"
        }
        success {
            echo "Build and push successful: ${IMAGE_NAME}"
        }
        failure {
            echo "Pipeline failed. Check the logs above."
        }
    }
}


