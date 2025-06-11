pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
        stage('Start Juice Shop') {
            steps {
                sh '''
                    docker rm -f juice-shop || true
                    docker run -d --name juice-shop -p 3000:3000 bkimminich/juice-shop
                    for i in {1..30}; do
                        curl -s http://localhost:3000/ >/dev/null && break
                        sleep 2
                    done
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
}