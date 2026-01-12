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

        // Clean npm cache to free memory
        sh 'npm cache clean --force'
        
        sh 'npm ci --prefer-offline'
        
        // Increased memory + disable source maps for faster builds
        sh '''
            export NODE_OPTIONS="--max-old-space-size=1536"
            export GENERATE_SOURCEMAP=false
            npm run build
        '''
        
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
            
            // 5. Build and Push (using -f to specify DockerFile with capital F)
            sh "docker build -f DockerFile -t ${imageTag} ."

            docker.withRegistry('https://index.docker.io/v1/', "dockerhub-creds") {
                docker.image(imageTag).push()
                docker.image(imageTag).push("latest")
            }
        }
    }
}
    
    stage('Update Deployment File') {
    agent any 
    environment {
        GIT_REPO_NAME = "react-k8s-gitops-pipeline"
        GIT_USER_NAME = "ShashmithaBan"
    }
    steps {
        // Use usernamePassword if that's how you stored your GitHub PAT
        withCredentials([usernamePassword(credentialsId: 'github-creds', 
                         usernameVariable: 'GIT_USER', 
                         passwordVariable: 'GIT_PASSWORD')]) {
            sh '''
                # Set local git config for this specific run
                git config user.email "gimansabandara2001@gmail.com"
                git config user.name "Shashmitha Bandara"
                
                # Faster sed execution
                sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" Manifests/Deployment.yml
                
                git add Manifests/Deployment.yml
                
                # Skip the commit if nothing changed to save time
                if git diff-index --quiet HEAD; then
                    echo "No changes to commit"
                else
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    # Use the PasswordVariable (Token) for the push
                    git push https://${GIT_USER}:${GIT_PASSWORD}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
                fi
            '''
        }
    }
}
}
}