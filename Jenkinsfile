```
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
                      -v $WORKSPACE/.zap:/zap/wrk \
                      -v $WORKSPACE/results:/zap/wrk/reports \
                      --network host \
                      ghcr.io/zaproxy/zaproxy:stable zap.sh -autorun /zap/wrk/passive.yaml
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
```
