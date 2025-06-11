pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Start Juice Shop') {
            steps {
                sh 'docker run -d --rm --name juice-shop -p 3000:3000 bkimminich/juice-shop'
            }
        }
        stage('ZAP passive scan') {
            steps {
                sh '''
                    mkdir -p results
                    docker run --rm --name zap \
                      --add-host host.docker.internal:host-gateway \
                      -v $WORKSPACE/.zap:/zap/wrk:ro \
                      -v $WORKSPACE/results:/zap/reports \
                      -w /zap/wrk \
                      ghcr.io/zaproxy/zaproxy:stable \
                      zap.sh -cmd -port 8090 -autorun passive.yaml
                '''
            }
        }
    }
    post {
        always {
            sh 'docker stop juice-shop || true'
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
        }
    }
}
