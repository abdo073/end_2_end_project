pipeline {
    agent any

    environment {
        LOG_DIR = "${WORKSPACE}/jenkins-logs"
    }

    stages {
        stage('Prepare Logs') {
            steps {
                script {
                    // إنشاء مجلد logs لو مش موجود
                    sh "mkdir -p ${LOG_DIR}"
                    
                    // نسخ ملفات build/logs لو محتاج
                    // sh "cp /var/log/jenkins/jenkins.log ${LOG_DIR}/"
                }
            }
        }

        stage('Security Scan - Wazuh Docker') {
            steps {
                script {
                    sh """
                    echo "Pulling Wazuh Docker image..."
                    docker pull wazuh/wazuh:4.14.2

                    echo "Running Wazuh container..."
                    docker run --rm -d --name wazuh-test \\
                        -p 1514:1514 \\
                        -v ${LOG_DIR}:/var/ossec/logs \\
                        wazuh/wazuh:4.14.2

                    # Optional: run a test scan inside container
                    echo "Running logtest inside Wazuh container..."
                    docker exec wazuh-test /var/ossec/bin/ossec-logtest < ${LOG_DIR}/jenkins.log

                    echo "Stopping Wazuh container..."
                    docker stop wazuh-test || true
                    """
                }
            }
        }

        stage('Post Scan') {
            steps {
                echo "Security scan finished. Check logs in ${LOG_DIR}."
            }
        }
    }
}
