pipeline {
    agent: any

    environment {
        DOCKER_HUB_USER = 'hexadec1malya'
        DOCKER_HUB_REPO = 'demo-app'
        EC2_USER = 'ubuntu'
        EC2_IP = '16.16.162.186'
        SSH_CRED_ID = 'ec2-ssh-key'
}
    stages {
        stage('Checkout'){
            steps {
                checkout scm
            }
        }
        stage('SAST With Bandit'){
            steps{
                script{
                  // Run scan with Bandit and generate report
                  sh '''
                    python3 -m venv bandit_venv
                    . bandit_venv/bin/activate
                    pip install --upgrade pip
                    pip install bandit

                    bandit -r . -f json -o bandit-report.json --exit-zero
                    bandit -r . -f json -o bandit-report.html --exit-zero
                    deactivate
                    '''
                   //Archive the report as artifact
                  archiveArtifacts artifacts: 'bandit-report.json, bandit-report.html', allowEmptyArchive: true

                  // Publish HTML Report
                    publishHTML(target: [
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
                                keepAll: true,
                                reportDir: '.',
                                reportFiles: 'bandit-report.html',
                                reportName: 'Bandit Security Report'
                            ])

                  // Parse JSON report to check for issues
                    script {
                        def jsonReport = readJSON file: 'bandit-report.json'
                        def issueCount = jsonReport.results.size()
                        if (issueCount > 0) {
                            echo "Bandit found ${issueCount} potential security issue(s). Please review the report."
                        } else {
                            echo "Bandit scan completed successfully with no issues found."
                        }
                    }
                }
              }
            }
        stage('Build & Push to Docker Hub') {
            steps {
                script {
                    // Log in using Jenkins Docker credentials (you'll need to set these up)
                    docker.withRegistry('', 'docker-hub-credentials-id') {
                        def appImage = docker.build("${DOCKER_HUB_USER}/${DOCKER_HUB_REPO}:${env.BUILD_NUMBER}")
                        appImage.push()
                        appImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                // This uses the "SSH Agent" plugin to handle your .pem key safely
                sshagent([SSH_CRED_ID]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                        docker pull ${DOCKER_HUB_USER}/${DOCKER_HUB_REPO}:latest
                        docker stop flask-app || true
                        docker rm flask-app || true
                        docker run -d --name flask-app -p 80:5000 ${DOCKER_HUB_USER}/${DOCKER_HUB_REPO}:latest
                    '
                    """
                }
            }
        }

        }

    }
}