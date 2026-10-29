pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 2, unit: 'HOURS')
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
    }

    parameters {
        string(
            name: 'DUT_HOST',
            defaultValue: '192.168.1.1',
            description: 'IP address of the router DUT'
        )
        string(
            name: 'DUT_BOARD',
            defaultValue: 'wnc-freedom',
            description: 'Board directory under .gitlab/tests/cram'
        )
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate prpl Cram suite') {
            steps {
                sh '''
                    set -eu
                    test -d .gitlab/tests/cram/generic
                    test -d ".gitlab/tests/cram/${DUT_BOARD}"
                    test -d .gitlab/tests/cram/post/generic
                    test -n "$(find .gitlab/tests/cram -name '*.t' -print -quit)"
                '''
            }
        }

        stage('Run prpl Cram suite') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'prpl-dut-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        set -eu
                        : "${DUT_HOST:?DUT_HOST is required}"
                        : "${DUT_BOARD:?DUT_BOARD is required}"
                        : "${SSH_USER:?Jenkins credential must provide SSH_USER}"
                        : "${SSH_KEY:?Jenkins credential must provide SSH_KEY}"

                        if ! python3 -m cram --version >/dev/null 2>&1; then
                            echo 'python3-cram is required on the Jenkins agent'
                            exit 1
                        fi

                        export CRAM_REMOTE_COMMAND="ssh -n -i ${SSH_KEY} -o BatchMode=yes -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${SSH_USER}@${DUT_HOST}"
                        export CRAM_REMOTE_COPY="scp -i ${SSH_KEY} -O -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null"

                        python3 -m cram --verbose \\
                            --xunit-file=cram-generic.xunit.xml \\
                            .gitlab/tests/cram/generic
                        python3 -m cram --verbose \\
                            --xunit-file=cram-${DUT_BOARD}.xunit.xml \\
                            ".gitlab/tests/cram/${DUT_BOARD}"
                        python3 -m cram --verbose \\
                            --xunit-file=cram-post.xunit.xml \\
                            .gitlab/tests/cram/post/generic
                    '''
                }
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: 'cram-*.xunit.xml'
            archiveArtifacts allowEmptyArchive: true, artifacts: 'cram-*.xunit.xml'
        }
        success {
            echo 'prpl Cram suite passed'
        }
        failure {
            echo 'prpl Cram suite failed; inspect the Cram and DUT logs'
        }
    }
}
