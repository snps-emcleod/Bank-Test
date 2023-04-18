def workspace
node {
    workspace = env.WORKSPACE
}
pipeline {
    agent any
    
    environment {
        CONNECT = 'https://us1a-eng-emcleod.nprd.sig.synopsys.com:8443'
        PROJECT = 'bankapp'
        BLDCMD = 'make'
        CHECKERS = '--webapp-security --enable-callgraph-metrics'
        COVERITY_NO_LOG_ENVIRONMENT_VARIABLES = '1'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                sh 'make'
                }
        }
        stage('Test') {
            steps {
                echo 'Testing not implemented'
            }
        }
        stage('Security Testing') {
            parallel {
                stage('Coverity Full Scan') {
                    steps {
                        withCoverityEnvironment(coverityInstanceUrl: "$CONNECT", projectName: "$PROJECT", streamName: "$PROJECT-$GIT_BRANCH") {
                            sh '''
                                cov-build --dir idir  $BLDCMD
                                cov-analyze --dir idir --strip-path $WORKSPACE $CHECKERS
                                cov-commit-defects --dir idir --ticker-mode none --url $COV_URL --stream $COV_STREAM \
                                    --description $BUILD_TAG --version $GIT_COMMIT
                            '''
                            script { // Coverity Quality Gate
                                count = coverityIssueCheck(viewName: 'OWASP Web Top 10', returnIssueCount: true)
                                if (count != 0) { unstable 'issues detected' }
                            }
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying not implemented'
            }
        }
    }
}