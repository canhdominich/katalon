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

        stage('Prepare Workspace') {
            steps {
                sh '''
                    echo "Checking workspace structure..."
                    ls -la

                    if [ ! -d "Test Suites" ]; then
                        echo "ERROR: Test Suites directory not found!"
                        exit 1
                    fi

                    if [ ! -d "TestData" ]; then
                        echo "ERROR: TestData directory not found!"
                        exit 1
                    fi

                    echo "Checking Excel files..."
                    ls -la TestData/

                    mkdir -p Reports
                '''
            }
        }

        stage('Execute Tests') {
            steps {
                sh '''
                    # Tải và cài đặt Katalon Studio
                    wget -q -O katalon.zip https://download.katalon.com/${KATALON_VERSION}/Katalon_Studio_Linux_64-${KATALON_VERSION}.zip
                    unzip -q katalon.zip -d katalon
                    chmod +x katalon/Katalon_Studio_Linux_64-${KATALON_VERSION}/katalon

                    # Chạy test
                    ./katalon/Katalon_Studio_Linux_64-${KATALON_VERSION}/katalon -noSplash -runMode=console -projectPath=${WORKSPACE} -retry=0 -testSuitePath="Test Suites/TSLogin" -browserType="Chrome (headless)" -executionProfile="default" -reportFolder=${WORKSPACE}/Reports -reportFileName="TestReport"
                '''
            }
        }

        stage('Publish Reports') {
            steps {
                script {
                    if (fileExists('Reports')) {
                        sh 'ls -la Reports/'

                        def reportPath = sh(script: 'find Reports -type d -name "TSLogin*" | sort -r | head -n 1', returnStdout: true).trim()
                        if (reportPath) {
                            echo "Found report directory at: ${reportPath}"
                            publishHTML([
                                allowMissing: true,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: reportPath,
                                reportFiles: '*.html',
                                reportName: 'Katalon Test Report'
                            ])
                        } else {
                            echo "No TSLogin directory found in Reports"
                        }
                    } else {
                        echo "Reports directory does not exist"
                    }
                }
            }
        }
    }
}
