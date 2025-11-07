pipeline {
    agent any

    tools {
        nodejs 'node18'  // Use NodeJS tool configured in Jenkins
    }

    environment {
        FTP_SERVER = '161.97.130.165'
        FTP_PORT = '21'
        FTP_PATH = '/'           // Root folder of IIS
        FTP_CREDENTIALS = credentials('ftp-credentials')
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/AnjumShekh/hello-mishainfotech.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing npm packages...'
                bat 'npm install'
            }
        }

        stage('Build React App') {
            steps {
                echo 'Building React project...'
                bat 'npm run build'
            }
        }

        stage('Deploy to FTP (IIS Root)') {
            steps {
                echo 'Deploying build files to IIS root via FTP...'
                bat '''
                    cd build
                    for /r %%F in (*) do (
                        echo Uploading %%F
                        curl -s -T "%%F" -u %FTP_CREDENTIALS_USR%:%FTP_CREDENTIALS_PSW% ftp://%FTP_SERVER%:%FTP_PORT%%FTP_PATH% --ftp-create-dirs
                    )
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Deployment failed!'
        }
    }
}
