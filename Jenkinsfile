pipeline {
    agent any
    tools {
        nodejs 'node20'
    }
    environment {
        SERVER_USER = 'root'
        SERVER_IP = '141.95.149.180'
    }
    stages {
        stage('Checks backend') {
            steps {
                dir('server') {
                    sh 'npm ci --cache .npm --prefer-offline'
                    sh 'npm run test:ci'
                    sh 'npm run lint'
                    sh 'npm audit'
                }
            }
        }
        stage('Checks frontend') {
            steps {
                dir('client') {
                    sh 'npm ci --cache .npm --prefer-offline'
                    sh 'npm audit'
                    sh 'npm run lint'
                }
            }
        }
        stage('Build frontend') {
            steps {
                dir('client') {
                    sh 'npm run build'
                }
            }
        }
        stage('Tests E2E sur Chrome') {
            steps {
                dir('server') {
                    sh 'node index.js &'
                }
                dir('client') {
                    sh 'BROWSER=chrome npm run test:e2e'
                }
            }
        }
        stage('Tests E2E sur Electron') {
            steps {
                dir('server') {
                    sh 'node index.js &'
                }
                dir('client') {
                    sh 'BROWSER=electron npm run test:e2e'
                }
            }
        }
        stage('Envoyer le Coverage frontend') {
            steps {
                withCredentials([string(credentialsId: 'codecov_token', variable: 'CODECOV_TOKEN')]) {
                    dir('server') {
                        sh '/home/jenkins/codecov -t ${CODECOV_TOKEN}'
                    }
                }
            }
        }
        stage('Déployer sur le serveur') {
            steps {
                sshagent(credentials: ['ovh_prod_key']) {
                    sh '''
                        [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                        ssh-keyscan -H $SERVER_IP >> ~/.ssh/known_hosts
                        rm -rf ./server/node_modules
                        scp -r ./client/dist/ $SERVER_USER@$SERVER_IP:/var/www/
                        scp -r ./server $SERVER_USER@$SERVER_IP:/var/www
                        ssh $SERVER_USER@$SERVER_IP "cd /var/www/server && npm install --omit=dev"
                        ssh $SERVER_USER@$SERVER_IP "cd /var/www/server && pm2 startOrRestart ecosystem.config.js --env production && pm2 save"
                    '''
                }
            }
        }
    }
}
