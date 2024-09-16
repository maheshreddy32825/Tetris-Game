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
            def creds = gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')
            if (creds) {
                withCredentials([creds]) {
                    def NEW_IMAGE_NAME = "mamir32825/tetrisv1:latest"
                    sh "sed -i 's|image: .*|image: ${NEW_IMAGE_NAME}|' deployment.yml"
                    sh 'git add .'
                    sh "git commit -m 'Update deployment image to ${NEW_IMAGE_NAME}'"
                    sh "git push https://${creds.username}:${creds.password}@github.com/${env.GIT_USER_NAME}/${env.GIT_REPO_NAME} HEAD:main"
                }
            } else {
                error "Failed to retrieve GitHub credentials."
            }
        }
    }
}

    }
    
}