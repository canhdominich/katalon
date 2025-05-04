pipeline {
    agent any

    environment {
        KATALON_VERSION = '10.2.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Execute Tests') {
            steps {
                executeKatalon(
                    version: env.KATALON_VERSION,
                    executeArgs: "-runMode=console -projectPath=${WORKSPACE} -retry=0 -testSuitePath='Test Suites/TSLogin' -browserType='Chrome (headless)' -executionProfile='default' -reportFolder=${WORKSPACE}/Reports -reportFileName='TestReport'"
                )
            }
        }

        stage('Publish Reports') {
            steps {
                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'Reports',
                    reportFiles: '*.html',
                    reportName: 'Katalon Test Report'
                ])
            }
        }
    }
}
