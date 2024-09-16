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
        NEW_IMAGE_NAME = "mamir32825/tetrisv1:latest"
        GITHUB_TOKEN = credentials('github')  
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
        		
		stage('Security Scans') {
		    parallel {
		        stage('OWASP Dependency-Check') {
		            steps {
		                // Perform OWASP Dependency-Check
		                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
		                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
		            }
		        }
		                
		        stage('TRIVY Filesystem Scan') {
		            steps {
		                // Perform TRIVY filesystem scan
		                sh "trivy fs . > trivyfs.txt"
		            }
		        }
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
                    //withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh "sed -i 's|image: .*|image: ${NEW_IMAGE_NAME}|' deployment.yml"
                    sh 'git add .'
                    sh "git commit -m 'Update deployment image to ${NEW_IMAGE_NAME}'"
                    sh "git push https://${GITHUB_TOKEN}@github.com/maheshreddy32825/Tetris-Game.git main"                  
                }
            }
        }
    }
    
}