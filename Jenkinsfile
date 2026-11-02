
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        VM1 = '192.168.100.10'
        DEPLOY_USER = 'trang_admin'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    test -s index.html
                    test -s Dockerfile
                '''
            }
        }

        stage('Package') {
            steps {
                sh 'tar -czf app.tar.gz index.html Dockerfile'
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ['vm1-deploy-ssh']) {
                    sh '''
                        scp -o BatchMode=yes app.tar.gz \
                          ${DEPLOY_USER}@${VM1}:/tmp/devops-app.tar.gz

                        ssh -o BatchMode=yes ${DEPLOY_USER}@${VM1} '
                            mkdir -p ~/cicd-deploy &&
                            tar -xzf /tmp/devops-app.tar.gz -C ~/cicd-deploy &&
                            cd ~/cicd-deploy &&
                            docker build -t devops-cicd:latest . &&
                            (docker rm -f devops-cicd-demo || true) &&
                            docker run -d \
                              --name devops-cicd-demo \
                              --restart unless-stopped \
                              -p 8084:80 \
                              devops-cicd:latest
                        '
                    '''
                }
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    curl -fsS --retry 5 --retry-delay 2 \
                      http://192.168.100.10:8084/
                '''
            }
        }
    }

    post {
        success {
            echo 'DEPLOYMENT SUCCESS'
        }

        failure {
            echo 'DEPLOYMENT FAILED'
        }
    }
}

