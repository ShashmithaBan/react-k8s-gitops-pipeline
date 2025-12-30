pipeline {
    agent none

    environment {
        DOCKER_IMAGE = "shashmitha2001/portfolio-deploy-react"
        SONAR_PROJECT_KEY = "Portfolio_2026"
        SONAR_ORG = "shashmithaban"
        SONAR_SERVER_ID   = "sonarcloud"
    }

    stages {

        stage('Checkout') {
            agent any
             steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/ShashmithaBan/Portfolio_2026.git'
                 }
        }

        stage('Install & Build React') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root'
                }
            }
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('SonarCloud Analysis') {
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli:latest'
                }
            }
            steps {
                // Automatically injects the Sonar token and URL from Jenkins config
               withSonarQubeEnv("${SONAR_SERVER_ID}") {
                    sh """
                      sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.organization=${SONAR_ORG} \
                        -Dsonar.sources=src
                    """
                }
            }
        }

        stage('Quality Gate') {
            agent any
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build & Push') {
    environment {
      
        DOCKER_IMAGE_TAG = "${DOCKER_IMAGE}:${BUILD_NUMBER}"
    }
    steps {
        script {
          
            sh "docker build -t ${DOCKER_IMAGE_TAG} ."

                      docker.withRegistry('https://index.docker.io/v1/', "dockerhub-creds") {
                def myImage = docker.image("${DOCKER_IMAGE_TAG}")
                myImage.push()
                
              

                myImage.push("latest")
            }
        }
    }
}
    }
    post {
        success {
            echo '✅ Build executed inside Docker containers'
        }
        failure {
            echo '❌ Pipeline failed'
        }
    }

}