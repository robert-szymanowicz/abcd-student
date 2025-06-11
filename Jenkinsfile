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
                    ls -la
                    mkdir -p results
                    pwd
                    echo $WORKSPACE
                    ls -la $WORKSPACE
                    ls -1 $WORKSPACE/.zap
                    ls -1 $WORKSPACE/results
                    cat $WORKSPACE/.zap/passive.yaml
                    docker run --rm --name zap \
                      --add-host host.docker.internal:host-gateway \
                      -v $WORKSPACE/.zap:/zap/wrk \
                      -v $WORKSPACE/results:/zap/wrk/reports \
                      -w /zap/wrk \
                      ghcr.io/zaproxy/zaproxy:stable \
                      zap.sh -cmd -port 8090 -autorun /zap/wrk/passive.yaml
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