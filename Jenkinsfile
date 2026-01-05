pipeline {
    agent none

    environment {
        DOCKER_IMAGE = "shashmitha2001/portfolio-deploy-react"
        SONAR_PROJECT_KEY = "Portfolio_2026"
        SONAR_ORG = "shashmithaban"
        SONAR_SERVER_ID   = "sonarcloud"
    }
    options {
            // This prevents Jenkins from overwriting your React code with the Jenkinsfile repo
            skipDefaultCheckout() 
    }
    stages {

            stage('Checkout') {
            agent any
            steps {
                cleanWs() // Deletes the old Jenkinsfile and package-lock.json leftovers
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/ShashmithaBan/Portfolio_2026.git'
            }
        }

        stage('Install & Build React') {
            agent {
                docker {
                    image 'node:20-alpine'
                    args '-u root'
                }
            }
            steps {
               // --no-audit and --prefer-offline make it much faster
        sh 'npm ci --prefer-offline'
        
        // LIMIT the memory to 512MB or 1GB to prevent the instance from crashing
        sh 'NODE_OPTIONS="--max-old-space-size=512" npm run build'
        stash name: 'build-output', includes: 'dist/**, Dockerfile'
            }
        }

        // stage('SonarCloud Analysis') {
        //     agent {
        //         docker {
        //             image 'sonarsource/sonar-scanner-cli:latest'
        //         }
        //     }
        //     steps {
        //         // Automatically injects the Sonar token and URL from Jenkins config
        //        withSonarQubeEnv("${SONAR_SERVER_ID}") {
        //             sh """
        //               sonar-scanner \
        //                 -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
        //                 -Dsonar.organization=${SONAR_ORG} \
        //                 -Dsonar.sources=src
        //             """
        //         }
        //     }
        // }

        // stage('Quality Gate') {
        //     agent any
        //     steps {
        //         timeout(time: 2, unit: 'MINUTES') {
        //             waitForQualityGate abortPipeline: true
        //         }
        //     }
        // }

stage('Docker Build & Push') {
    // This is the missing piece that provides the 'FilePath' context
    agent any 

    steps {
        script {
            // Unstash the build files from the previous stage
            unstash 'build-output'
            
            // Define your image tag
            def imageTag = "${DOCKER_IMAGE}:${BUILD_NUMBER}"
            
            // Build the image
            sh "docker build -t ${imageTag} ."

            // Login and Push to Docker Hub
            docker.withRegistry('https://index.docker.io/v1/', "dockerhub-creds") {
                def myImage = docker.image(imageTag)
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
