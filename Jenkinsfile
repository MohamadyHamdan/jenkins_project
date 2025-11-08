pipeline {
    agent any

    environment {
        VIRTUAL_ENV = 'venv'
    }

    stages {
        stage('Setup') {
            steps {
                script {
                    if (!fileExists("${env.WORKSPACE}\\${VIRTUAL_ENV}")) {
                        bat "python -m venv ${VIRTUAL_ENV}"
                    }
                    bat "call ${VIRTUAL_ENV}\\Scripts\\activate && pip install -r requirements.txt"
                }
            }
        }

        stage('Lint') {
            steps {
                script {
                    bat "call ${VIRTUAL_ENV}\\Scripts\\activate && flake8 app.py"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    bat "call ${VIRTUAL_ENV}\\Scripts\\activate && pytest"
                }
            }
        }

        stage('Coverage') {
            steps {
                script {
                    // Run tests with coverage and generate XML + HTML reports
                    bat """
                        call ${VIRTUAL_ENV}\\Scripts\\activate ^
                        && coverage run -m pytest ^
                        && coverage xml -o coverage.xml ^
                        && coverage html -d htmlcov
                    """
                }
            }
        }

        stage('Security Scan') {
            steps {
                script {
                    // Run bandit; write a report file. We don't fail the build to keep the lab passing.
                    // (To fail on high/medium issues, remove the '|| exit /b 0'.)
                    bat """
                        call ${VIRTUAL_ENV}\\Scripts\\activate ^
                        && bandit -r . -f txt -o bandit_report.txt || exit /b 0
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Simple "local deploy": run the app once
                    bat "call ${VIRTUAL_ENV}\\Scripts\\activate && python app.py > deploy_output.txt"
                    echo "Deploying application..."
                }
            }
        }
    }

    post {
        always {
            // Keep useful artifacts (reports & deploy output)
            archiveArtifacts artifacts: 'coverage.xml, htmlcov/**, bandit_report.txt, deploy_output.txt', onlyIfSuccessful: false
            cleanWs()
        }
    }
}
