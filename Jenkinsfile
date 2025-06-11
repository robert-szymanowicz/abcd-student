pipeline {
    agent any
    stages {
        stage('Start Juice Shop') {
            steps {
                sh 'docker run -d --name juice-shop -p 3000:3000 bkimminich/juice-shop'
            }
        }
        stage('ZAP Passive Scan') {
            steps {
                sh '''
                    mkdir -p results
                    docker run --rm \
                        --add-host=host.docker.internal:host-gateway \
                        -v $PWD/.zap:/zap/rules:ro \
                        -v $PWD/results:/zap/wrk:rw \
                        ghcr.io/zaproxy/zaproxy:stable \
                        zap-baseline.py \
                        -t http://host.docker.internal:3000 \
                        -c /zap/rules/passive.yaml \
                        -r /zap/wrk/zap-report.html
                '''
            }
        }
        stage('Stop Juice Shop') {
            steps {
                sh 'docker rm -f juice-shop || true'
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
        }
    }
}


