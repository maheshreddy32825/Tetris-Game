pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    
    environment {
        // define env
        SCANNER_HOME = tool 'sonar-scanner'
        GIT_REPO_NAME = "Tetris-deployment-file"
        GIT_USER_NAME = "maheshreddy32825"
        DOCKER_IMAGE_NAME = 'tetrisv1'
    }
    
    stages {
        stage('Clean workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/maheshreddy32825/Tetris-Game.git'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectName=tetris -Dsonar.projectKey=tetris"
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                    // Wait for SonarQube Quality Gate
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script {
                    // Build Docker image
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build -t tetrisv1 ."
                        sh "docker tag tetrisv1 mamir32825/tetrisv1:latest"
                        sh "docker push mamir32825/tetrisv1:latest"
                    }
                }
            }
        }

        stage('Update Deployment File') {
            steps {
                script {
                    // Retrieve GitHub credentials
                    def creds = credentials('github')
                    
                    if (creds) {
                        withCredentials([usernamePassword(credentialsId: creds.id, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                            // Define the new image name
                            def NEW_IMAGE_NAME = "mamir32825/tetrisv1:latest"
                            
                            // Debug print to verify the values are correctly set
                            echo "NEW_IMAGE_NAME: ${NEW_IMAGE_NAME}"
                            echo "GIT_USER_NAME: ${env.GIT_USER_NAME}"
                            echo "GIT_REPO_NAME: ${env.GIT_REPO_NAME}"
                            
                            // Update deployment.yml with the new image name
                            sh "sed -i 's|image: .*|image: ${NEW_IMAGE_NAME}|' deployment.yml"
                            
                            // Stage and commit changes
                            sh 'git add .'
                            sh "git commit -m 'Update deployment image to ${NEW_IMAGE_NAME}'"
                            
                            // Push changes to GitHub
                            sh "git push https://${USERNAME}:${PASSWORD}@github.com/${env.GIT_USER_NAME}/${env.GIT_REPO_NAME} HEAD:main"
                        }
                    } else {
                        error "Failed to retrieve GitHub credentials."
                    }
                }
            }
        }
    }
    
}