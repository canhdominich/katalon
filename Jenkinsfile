pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Execute Tests') {
            steps {
                executeKatalon(
                    testSuitePath: 'Test Suites/TSLogin',
                    browserType: 'Chrome (headless)',
                    executionProfile: 'default',
                    reportFolder: 'Reports',
                    reportFileName: 'TestReport'
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
