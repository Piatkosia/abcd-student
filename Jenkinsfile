pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-pat', url: 'https://github.com/Piatkosia/abcd-student.git', branch: 'main'
                }
            }
         }

        stage('Example') {
            steps {
                echo 'Hello!'
                sh 'ls -la'
            }
        }

       stage('Prepare') {
            steps {
                sh 'mkdir -p results/'
            }
        }


          stage('OSV') {
            steps {
                sh '''
                osv-scanner scan --lockfile package-lock.json --json --output "${WORKSPACE}/results/osv-report.json"  || true
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
                    defectDojoPublisher(artifact: 'results/osv-report.json', productName: 'Juice Shop', scanType: 'OSV Scan', engagementName: 'piatkosia.apt@interia.pl')
                }
            }
        }

        stage('Trufflehog') {
            steps {
                sh '''
                    trufflehog git file://. --branch main --only-verified --json > "${WORKSPACE}/results/trufflehog-results.json"
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
                    defectDojoPublisher(artifact: 'results/trufflehog-results.json', productName: 'Juice Shop', scanType: 'Trufflehog Scan', engagementName: 'piatkosia.apt@interia.pl')
                }
            }
        }

        stage ('SEMGREP') {
            steps {
                sh 'semgrep scan --config auto --json > results/semgrep-results.json'
            }
            post {
                always {
                    echo 'Archiving results'
                    archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
                    echo 'Sending SEMGREP scan report to DefectDojo'
                    defectDojoPublisher(artifact: 'results/semgrep-results.json', productName: 'Juice Shop', scanType: 'Semgrep JSON Report', engagementName: 'piatkosia.apt@interia.pl')
                }
            }
        }


        stage('[ZAP]ierdala pasywnie') {
            steps {
                sh '''
                docker stop zap juice-shop || true
                    docker run --name juice-shop -d --rm \
                        -p 4000:3000 bkimminich/juice-shop
                    sleep 32
                '''
                sh '''
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v C:/Users/piatk/source/repos/abcd-student/.zap:/zap/wrk/:rw  \
                       -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                       "mkdir -p /zap/wrk/reports; zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml --user root" \
                                    || true
                '''
            }
            post {
                always {
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html "${WORKSPACE}/results/zap_html_report.html"
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml "${WORKSPACE}/results/zap_xml_report.xml"
                        docker stop zap
                        docker rm zap
                    '''
                    echo 'Budowanie artefakt�w'
                    archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
                    echo 'Zobaczymy czy DefectDojo przepu�ci'
                    defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: 'piatkosia.apt@interia.pl')
                }
            }
        }
    }
}