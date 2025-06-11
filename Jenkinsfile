pipeline {
    agent any           // built-in (master) node

    stages {
        /* … previous stages: build, tests, etc. … */

        stage('ZAP Passive Scan') {
            steps {
                script {
                    // 1. Directory for reports
                    sh 'mkdir -p results'

                    // 2. Launch the target application (Juice Shop)
                    sh '''
                      docker run -d --name juice-shop \
                        -p 3000:3000 \
                        --rm \
                        bkimminich/juice-shop
                    '''

                    // 3. Run ZAP passive scan (zap-baseline.py)
                    //    - host network so that ZAP can reach http://host.docker.internal:3000
                    //    - results and your .zap configuration are mounted under /zap/wrk
                    sh '''
                      docker run --rm --name zap \
                        --network host \
                        -v $PWD/results:/zap/wrk \
                        -v $PWD/.zap:/zap/wrk/.zap:ro \
                        ghcr.io/zaproxy/zaproxy:stable \
                        zap-baseline.py \
                          -t http://host.docker.internal:3000 \
                          -c /zap/wrk/.zap/passive.yaml \   # your disabled/enforced rules
                          -w zap_report.md \               # Markdown
                          -x zap_report.xml \              # XML (ZAP NIST schema)
                          -j -J zap_report.json \          # JSON + JUnit
                          -r zap_report.html \             # pretty HTML
                          -I -m 3                          # ignore exit status, crawl limit 3 min
                    '''
                }
            }
            post {
                always {
                    echo 'Stopping Juice Shop…'
                    sh 'docker stop juice-shop || true'

                    echo 'Archiving ZAP results…'
                    archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
                }
            }
        }
    }
}
