pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Start Juice Shop') {
            steps {
                sh '''
                    docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                    sleep(60)
                '''
            }
        }
        stage('ZAP passive scan') {
            steps {
                sh '''
                    docker run --rm --name zap \
                      --add-host=host.docker.internal:host-gateway \
                      -v /home/krot2ks/abcd-lab/abcd-student/.zap:/zap/wrk \
                      -v /home/krot2ks/abcd-lab/results:/zap/wrk/reports \
                      -w /zap/wrk \
                      ghcr.io/zaproxy/zaproxy:stable \
                      zap.sh -cmd -port 8090 -autorun /zap/wrk/passive.yaml
                '''
            }
        }
    }
    post {
        always {
            sh 'docker rm -f juice-shop || true'
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
        }
    }
}0)