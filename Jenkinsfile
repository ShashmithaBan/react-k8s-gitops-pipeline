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
            image 'node:20-slim'
            args '-u root'
        }
    }
    steps {
        // Force checkout to the CURRENT directory
        checkout([$class: 'GitSCM', 
            branches: [[name: 'main']], 
            userRemoteConfigs: [[url: 'https://github.com/ShashmithaBan/Portfolio_2026.git', credentialsId: 'github-creds']]
        ])

        sh 'npm ci --prefer-offline'
        sh 'NODE_OPTIONS="--max-old-space-size=512" npm run build'
        
        script {
            // This will show us EVERY file in the workspace to debug
            sh 'ls -R' 
            
            // If it still can't find it, we search for it
            sh 'find . -name "Dockerfile"'
        }
        
        // Use a wildcard to find the Dockerfile wherever it is in the root
        stash name: 'build-output', includes: 'dist/**, **/Dockerfile'
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
    agent any 
    steps {
        script {
            // 1. Clear the workspace to avoid permission issues
            cleanWs() 

            // 2. Pull the code repo directly to THIS agent
            // This ensures the Dockerfile is physically present in the root
            checkout([$class: 'GitSCM', 
                branches: [[name: 'main']], 
                userRemoteConfigs: [[url: 'https://github.com/ShashmithaBan/Portfolio_2026.git', credentialsId: 'github-creds']]
            ])
// 1. Show ALL files and folders recursively
    sh 'ls -R' 

    // 2. Search for the file regardless of case
    sh 'find . -iname "dockerfile"'
            // 3. Bring in the 'dist' folder from the previous build stage
            unstash 'build-output'

            

            def imageTag = "shashmitha2001/portfolio-deploy-react:${BUILD_NUMBER}"
            
            // 5. Build and Push
            sh "docker build -t ${imageTag} ."

            docker.withRegistry('https://index.docker.io/v1/', "dockerhub-creds") {
                docker.image(imageTag).push()
                docker.image(imageTag).push("latest")
            }
        }
    }
}
    }
   post {
        always {
          node('built-in') {
            script {
                // Remove local images to save disk space
                sh "docker rmi ${DOCKER_IMAGE}:${BUILD_NUMBER} || true"
                sh "docker rmi ${DOCKER_IMAGE}:latest || true"
                
                // Fully wipe the workspace to prevent stash conflicts
                cleanWs()
            }
          }
        }
    }

}
