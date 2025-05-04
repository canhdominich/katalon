pipeline {
    agent any

    environment {
        KATALON_VERSION = '10.2.0'
        KATALON_KEY = 'fb4e1d81-f3d3-4190-9474-a37ce9801ad1'
        CHROME_BIN = '/usr/bin/google-chrome-stable'
    }

    stages {
        stage('Setup Environment') {
            steps {
                sh '''
                    # Update package list
                    apt-get update
                    
                    # Install Chrome and dependencies
                    apt-get install -y wget gnupg2
                    wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add -
                    echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google.list
                    apt-get update
                    apt-get install -y google-chrome-stable xvfb
                    
                    # Verify Chrome installation
                    google-chrome-stable --version
                    which google-chrome-stable
                '''
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

                    mkdir -p Reports
                '''
            }
        }

        stage('Execute Tests') {
            steps {
                script {
                    try {
                        sh '''
                            # Start Xvfb
                            Xvfb :99 -screen 0 1280x1024x24 &
                            export DISPLAY=:99
                        '''
                        
                        executeKatalon(
                            version: env.KATALON_VERSION,
                            executeArgs: "-runMode=console -projectPath=${WORKSPACE} -retry=0 -testSuitePath='Test Suites/TSLogin' -browserType='Chrome (headless)' -executionProfile='default' -reportFolder=${WORKSPACE}/Reports -reportFileName='TestReport' -apikey=${env.KATALON_KEY} --config -webui.autoUpdateDrivers=true -webui.chrome.driverPath=/usr/bin/chromedriver"
                        )
                    } catch (Exception e) {
                        echo "Test execution failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        throw e
                    } finally {
                        sh '''
                            # Cleanup Xvfb
                            pkill Xvfb
                        '''
                    }
                }
            }
        }

        stage('Publish Reports') {
            steps {
                script {
                    if (fileExists('Reports')) {
                        sh 'ls -la Reports/'

                        def reportPath = sh(script: 'find Reports -type d -name "TSLogin" | head -n 1', returnStdout: true).trim()
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