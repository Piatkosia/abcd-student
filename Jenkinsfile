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

        stage('[ZAP]ierdala pasywnie') {
            steps {
                sh '''
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 bkimminich/juice-shop
                    sleep 32
                '''
                sh '''
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v C:\Users\piatk\source\repos\abcd-student\.zap:/zap/wrk/:rw  \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \
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
                    echo 'Budowanie artefaktów'
                    archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
                    echo 'Zobaczymy czy DefectDojo przepuœci'
                    defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: 'piatkosia.apt@interia.pl')
                }
            }
        }
    }
