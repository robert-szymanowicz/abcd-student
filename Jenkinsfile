// pipeline {
//     agent any
//     options {
//         skipDefaultCheckout(true)
//     }
//     stages {
//         stage('Start Juice Shop') {
//             steps {
//                 sh 'docker rm -f juice-shop || true'
//                 sh 'docker run -d --name juice-shop -p 3000:3000 bkimminich/juice-shop'
//                 sleep time: 60, unit: 'SECONDS'
//             }
//         }
//         stage('ZAP passive scan') {
//             steps {
//                 sh '''
//                     docker run --rm --name zap \
//                       --add-host=host.docker.internal:host-gateway \
//                       -v /home/krot2ks/abcd-lab/abcd-student/.zap:/zap/wrk \
//                       -v /home/krot2ks/abcd-lab/results:/zap/wrk/reports \
//                       -w /zap/wrk \
//                       ghcr.io/zaproxy/zaproxy:stable \
//                       zap.sh -cmd -port 8090 -autorun /zap/wrk/passive.yaml
//                 '''
//             }
//         }
//     }
//     post {
//         always {
//             sh 'docker rm -f juice-shop || true'
//             archiveArtifacts artifacts: '/home/krot2ks/abcd-lab/results/*', fingerprint: true, allowEmptyArchive: true
//         }
//     }
// }
pipeline {
    agent { label 'built-in' }

    stages {
        stage('SCA scan') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    sh '''
                        mkdir -p results
                        osv-scanner scan --lockfile package-lock.json --format json --output results/sca-osv-scanner.json
                    '''
                }
            }
        }

    stage('Secrets scan') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    sh '''
                        mkdir -p results
                        trufflehog git --branch main --repo . --json --fail --output results/secrets-trufflehog.json
                    '''
                }
            }
        }
    }
}

    post {
        always {
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
        }
    }
}
