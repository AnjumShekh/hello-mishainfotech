pipeline {
    agent any

    environment {
        FTP_SERVER = '161.97.130.165'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/AnjumShekh/hello-mishainfotech'
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
                    echo "📦 Uploading build folder recursively to FTP..."

                    // Use Jenkins credentials securely
                    withCredentials([usernamePassword(credentialsId: 'ftp-credentials', usernameVariable: 'FTP_USER', passwordVariable: 'FTP_PASS')]) {
                        bat '''
                            cd build
                            for /R %%F in (*) do (
                                curl -s -T "%%F" -u %FTP_USER%:%FTP_PASS% ftp://161.97.130.165:21/ --ftp-create-dirs
                            )
                        '''
                    }
