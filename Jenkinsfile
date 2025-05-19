pipeline {
    agent any
    environment {
        DOCKER_CREDENTIALS_ID = '0f6800e7-179e-452d-a666-14135c0ff0cf' 
        BACKEND_IMAGE = 'ashyrmalik/taskmgr-backend'
        FRONTEND_IMAGE = 'ashyrmalik/taskmgr-frontend' 
        COMPOSE_FILE = 'docker-compose-jenkins.yml' 
    }
    
    stages {
        stage('Git Clone') {
            steps {
                git url: 'https://github.com/AshyrMalik/TM.git', 
                     branch: 'main'
                echo 'Code cloned successfully 📦'
            }
        }
        
        stage('Build Images') {
            steps {
                script {
                    // Build backend
                    docker.build("${BACKEND_IMAGE}", './backend') 
                    
                    // Build frontend (if needed, or use pre-built image)
                    docker.build("${FRONTEND_IMAGE}", './client') 
                    
                    echo '✅ Images built successfully'
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: env.DOCKER_CREDENTIALS_ID,
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        // Push backend
                        docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_USER}", "${DOCKER_PASS}") {
                            docker.image("${BACKEND_IMAGE}").push('latest')
                        }
                        
                        // Push frontend
                        docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_USER}", "${DOCKER_PASS}") {
                            docker.image("${FRONTEND_IMAGE}").push('latest')
                        }
                        
                        echo '📤 Images pushed to Docker Hub'
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // Use different project name and compose file
                    sh 'docker-compose -p jenkins-taskmanager -f ${COMPOSE_FILE} down'
                    sh 'docker-compose -p jenkins-taskmanager -f ${COMPOSE_FILE} up -d'
                    echo '🚀 Containers deployed successfully'
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
            echo '🧹 Workspace cleaned'
        }
    }
}
