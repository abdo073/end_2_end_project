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

      stage('Run Wazuh Docker Stack') {
            steps {
                script {
                    sh """
                    docker pull wazuh/wazuh-indexer:4.3.10
                    docker pull wazuh/wazuh-manager:4.3.10
                    docker pull wazuh/wazuh-dashboard:4.3.10

                    docker network create wazuh-net || true
                    docker run -d --name wazuh-indexer --network wazuh-net wazuh/wazuh-indexer:4.3.10
                    docker run -d --name wazuh-manager --network wazuh-net -v ${LOG_DIR}:/var/ossec/logs wazuh/wazuh-manager:4.3.10
                    docker run -d --name wazuh-dashboard --network wazuh-net -p 5601:5601 wazuh/wazuh-dashboard:4.3.10

                    sleep 30
                    docker exec wazuh-manager /var/ossec/bin/ossec-logtest < ${LOG_DIR}/sample.log

                    docker stop wazuh-indexer wazuh-manager wazuh-dashboard
                    docker rm wazuh-indexer wazuh-manager wazuh-dashboard
                    """
                }
            }
        }
    }
}
