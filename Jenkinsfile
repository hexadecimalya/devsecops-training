pipeline {
    agent any // Removed the colon

    environment {
        DOCKER_HUB_USER = 'hexadec1malya'
        DOCKER_HUB_REPO = 'demo-app'
        EC2_USER        = 'ubuntu'
        EC2_IP          = '16.16.162.186'
        SSH_CRED_ID     = 'ec2-ssh-key'
        DOCKER_CREDS    = 'docker-hub-credentials-id'
        TAG             = 'latest'

    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

//        stage('SAST With Bandit') {
//            steps {
//                script {
//                    sh '''
//                        python3 -m venv bandit_venv
//                        . bandit_venv/bin/activate
//                        pip install --upgrade pip
//                        pip install bandit
//                        bandit -r . --exclude bandit_venv,venv,.venv -f json -o bandit-report.json --exit-zero
//                        bandit -r . --exclude bandit_venv,venv,.venv -f html -o bandit-report.html --exit-zero
//                        deactivate
//                    '''
//
//                    archiveArtifacts artifacts: 'bandit-report.json, bandit-report.html', allowEmptyArchive: true
//
//
//                    publishHTML(target: [
//                        allowMissing: false,
//                        alwaysLinkToLastBuild: false,
//                        keepAll: true,
//                        reportDir: '.',
//                        reportFiles: 'bandit-report.html',
//                        reportName: 'Bandit Security Report'
//                    ])
//
//                    // 4. Parse JSON
//                    def jsonReport = readJSON file: 'bandit-report.json'
//                    def issueCount = jsonReport.results.size()
//                    if (issueCount > 0) {
//                        echo "Bandit found ${issueCount} potential security issue(s). Check the tab in Jenkins!"
//                    } else {
//                        echo "Bandit scan completed with no issues."
//                    }
//                }
//            }
//        }

//        stage('Build & Push to Docker Hub') {
//            steps {
//                script {
//                    echo "before first docker command"
//                    docker.withRegistry('', "${DOCKER_CREDS}") {
//                        def appImage = docker.build("${DOCKER_HUB_USER}/${DOCKER_HUB_REPO}:${env.BUILD_NUMBER}")
//                        appImage.push()
//                        appImage.push('latest')
//                    }
//                }
//            }
//        }

        stage('Deploy to EC2') {
            steps {
                sshagent([SSH_CRED_ID]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                        cd ~/app &&
                        docker pull ${DOCKER_HUB_USER}/${DOCKER_HUB_REPO}:latest &&
                        docker build -t ${DOCKER_HUB_REPO}:latest . &&
                        docker push ${DOCKER_HUB_REPO}:latest &&
                        docker stop demo-app || true &&
                        docker rm demo-app || true &&
                        docker run -d --name demo-app -p 80:5000 ${DOCKER_HUB_USER}/${DOCKER_HUB_REPO}:latest
                    '
                    """
                }
            }
        }
    }
}