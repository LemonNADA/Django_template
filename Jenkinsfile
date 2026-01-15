pipeline {
    agent any

    parameters {
        string(name: 'STUDENT_NAME', defaultValue: 'cheptsov', description: 'Имя ученика для поддомена')
        string(name: 'STUDENT_PORT', defaultValue: '18013', description: 'Порт на сервере')
        string(name: 'DOMAIN', defaultValue: 'mshptop.ru', description: 'Базовый домен')
    }

    environment {
        DOCKER_HOST = "unix:///var/run/docker.sock"
        IMAGE_NAME = "django_demo_app:${params.STUDENT_NAME}"
        CONTAINER_NAME = "django_demo_app_${params.STUDENT_NAME}"
    }


    stages {
        stage('Check workspace') {
            steps {
                sh '''
                    echo "Current directory:"
                    pwd
                    echo "Files in workspace:"
                    ls -la
                    echo "Checking Dockerfile..."
                    ls -la Dockerfile || echo "Dockerfile not found!"
                '''
            }
        }

        stage('Cleanup old containers') {
            steps {
                script {
                    sh """
                        # Останавливаем и удаляем старый контейнер если существует
                        docker rm -f ${CONTAINER_NAME} 2>/dev/null || echo "Container ${CONTAINER_NAME} not found"
                    """
                }
            }
        }

        stage('Deploy app (docker)') {
            steps {
                script {
                    sh """
                        echo ""
                        echo "Edge nginx target:"
                        echo "  https://${params.STUDENT_NAME}.${params.DOMAIN} -> http://127.0.0.1:${params.STUDENT_PORT}"
                        echo ""
                        
                        # Проверяем доступность Docker
                        docker version
                        
                        # Собираем образ
                        echo "Building Docker image..."
                        docker build -t ${IMAGE_NAME} .

                        # Запускаем контейнер
                        echo "Starting container..."
                        docker run -d \\
                            --name ${CONTAINER_NAME} \\
                            -p ${params.STUDENT_PORT}:8000 \\
                            -e SECRET_KEY=dev-secret-key \\
                            ${IMAGE_NAME}
                        
                        # Проверяем, что контейнер запустился
                        echo "Container status:"
                        docker ps --filter "name=${CONTAINER_NAME}"
                        
                        # Проверяем логи (первые 5 строк)
                        echo "Container logs (first 5 lines):"
                        docker logs --tail 5 ${CONTAINER_NAME} 2>/dev/null || echo "No logs yet"
                    """
                }
            }
        } 

        stage('Smoke check') {
            steps {
                script {
                    sh """
                        echo "Waiting for app to start..."
                        sleep 10
                        
                        # Пробуем подключиться к приложению
                        MAX_RETRIES=10
                        RETRY_COUNT=0
                        
                        while [ \$RETRY_COUNT -lt \$MAX_RETRIES ]; do
                            echo "Attempt \$((RETRY_COUNT + 1))/\$MAX_RETRIES..."
                            
                            if curl -f --max-time 10 http://127.0.0.1:${params.STUDENT_PORT}/ 2>/dev/null; then
                                echo "✓ App is responding successfully!"
                                echo "HTTP Response:"
                                curl -s http://127.0.0.1:${params.STUDENT_PORT}/ | head -20
                                exit 0
                            fi
                            
                            RETRY_COUNT=\$((RETRY_COUNT + 1))
                            echo "App not ready yet, waiting 5 seconds..."
                            sleep 5
                        done
                        
                        echo "✗ ERROR: App failed to start or respond after \$MAX_RETRIES attempts"
                        echo "Container logs (last 30 lines):"
                        docker logs --tail 30 ${CONTAINER_NAME} 2>/dev/null || echo "Failed to get logs"
                        exit 1
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed"
            sh """
                echo "=== FINAL STATUS ==="
                echo "Container:"
                docker ps --filter "name=${CONTAINER_NAME}" 2>/dev/null || echo "Container not running"
                echo ""
                echo "Port ${params.STUDENT_PORT} usage:"
                ss -tlnp | grep ":${params.STUDENT_PORT}" || echo "Port ${params.STUDENT_PORT} is free"
            """
        }
        failure {
            echo 'Pipeline failed!!! :('
            sh """
                echo "=== DEBUG INFO ==="
                echo "Docker images:"
                docker images ${IMAGE_NAME} 2>/dev/null || echo "Image not found"
                echo ""
                echo "Recent Docker events:"
                docker events --since 5m 2>/dev/null | tail -20 || echo "Could not get events"
                echo ""
                echo "Container inspect (if exists):"
                docker inspect ${CONTAINER_NAME} 2>/dev/null | head -100 || echo "Container not found"
            """
        }
        success {
            echo 'Pipeline succeeded! ✓'
            sh """
                echo "Application deployed successfully!"
                echo "Access URL: https://${params.STUDENT_NAME}.${params.DOMAIN}"
                echo "Local URL: http://127.0.0.1:${params.STUDENT_PORT}"
            """
        }
    }
} 