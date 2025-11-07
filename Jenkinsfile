pipeline {
    agent any

    environment {
        FTP_SERVER = '161.97.130.165'
        FTP_USER = 'your_ftp_username'
        FTP_PASSWORD = 'your_ftp_password'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/AnjumShekh/your-react-repo.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('Build React App') {
            steps {
                bat 'npm run build'
            }
        }

        stage('Deploy to IIS via FTP') {
            steps {
                script {
                    echo "📦 Uploading build folder recursively..."
                    bat '''
                        cd build
                        for /r %%F in (*) do (
                            curl -s -T "%%F" -u %FTP_USER%:%FTP_PASSWORD% ftp://%FTP_SERVER%:21/ --ftp-create-dirs
                        )
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}
