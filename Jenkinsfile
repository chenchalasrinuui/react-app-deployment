pipeline {
    agent any

    environment {
        FTP_SERVER = '148.72.90.37'          // ← replace with your real Plesk IP
        REMOTE_DIR = '/dev.uijavakit.com/'   // ← replace with your subdomain path
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install & Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }

        stage('Deploy via FTP') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'godaddy-ftp-creds',
                    usernameVariable: 'FTP_USER',
                    passwordVariable: 'FTP_PASS'
                )]) {
                    sh '''
                        lftp -u "$FTP_USER,$FTP_PASS" "$FTP_SERVER" <<EOF
set ssl:verify-certificate no
mirror --reverse --delete --verbose dist/ "$REMOTE_DIR" \
  --exclude-glob .well-known/ \
  --exclude-glob error_log \
  --exclude-glob logs/
bye
EOF
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployed dist/ to ${REMOTE_DIR} on ${FTP_SERVER}"
        }
        failure {
            echo "Deploy failed — check the FTP stage log above."
        }
    }
}