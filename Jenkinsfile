pipeline {
    agent any
    parameters {
        choice(
            name: 'BACKEND_COUNT', 
            choices: ['1', '2'], 
            description: 'Number of backend containers to deploy'
        )
    }
    stages {
        stage('Build Backend Image') {
            steps {
                sh '''
                # Go to backend folder
                cd $WORKSPACE/backend || exit 1

                # Remove old Docker image if exists
                docker rmi -f backend-app || true

                # Build Docker image
                docker build -t backend-app .
                '''
            }
        }
        stage('Deploy Backend Containers') {
            steps {
                sh '''
                # Create Docker network if not exists
                docker network create app-network || true

                # Remove old backend containers
                for i in 1 2; do
                    docker rm -f backend$i || true
                done

                # Run backend containers based on parameter
                for i in $(seq 1 $BACKEND_COUNT); do
                    docker run -d --name backend$i --network app-network backend-app
                done
                '''
            }
        }
        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                # Remove old NGINX container
                docker rm -f nginx-lb || true

                # Run NGINX with mounted custom config
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  -v $WORKSPACE/nginx/default.conf:/etc/nginx/conf.d/default.conf:ro \
                  nginx
                '''
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
