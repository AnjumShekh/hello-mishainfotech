pipeline {
    agent any

    environment {
        FTP_SERVER = 'ftp://161.97.130.165'
        LOCAL_DIR = 'build'
        REMOTE_DIR = '/'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/AnjumShekh/hello-mishainfotech.git'
            }
        }

        stage('Build React App') {
            steps {
                echo '🏗️ Installing dependencies and building React app...'
                bat 'npm install'
                bat 'npm run build'
            }
        }

        stage('Deploy to FTP') {
            steps {
                script {
                    echo "📦 Uploading ${env.LOCAL_DIR} → ${env.REMOTE_DIR} on ${env.FTP_SERVER}"

                    withCredentials([usernamePassword(credentialsId: 'ftp-credentials', usernameVariable: 'FTP_USER', passwordVariable: 'FTP_PASS')]) {
                        powershell """
                            \$sourcePath = "${env.LOCAL_DIR}"
                            \$ftpServer = "${env.FTP_SERVER}"
                            \$creds = New-Object System.Net.NetworkCredential(\$env:FTP_USER, \$env:FTP_PASS)

                            function New-FtpDirectory {
                                param(\$ftpUrl, \$credentials)
                                try {
                                    \$req = [System.Net.FtpWebRequest]::Create(\$ftpUrl)
                                    \$req.Credentials = \$credentials
                                    \$req.Method = [System.Net.WebRequestMethods+Ftp]::MakeDirectory
                                    \$req.GetResponse().Close()
                                    Write-Host "📁 Created folder: \$ftpUrl"
                                } catch {
                                    # Ignore if folder exists
                                }
                            }

                            Get-ChildItem -Path \$sourcePath -Recurse | ForEach-Object {
                                if (-not \$_.PSIsContainer) {
                                    \$relativePath = \$_.FullName.Substring(\$sourcePath.Length + 1).Replace('\\\\', '/')
                                    \$remoteDir = [System.IO.Path]::GetDirectoryName(\$relativePath).Replace('\\\\', '/')

                                    if (\$remoteDir -ne '') {
                                        \$ftpDirUrl = "\$ftpServer/\$remoteDir"
                                        New-FtpDirectory -ftpUrl \$ftpDirUrl -credentials \$creds
                                    }

                                    \$uri = "\$ftpServer/\$relativePath"
                                    Write-Host "⬆️ Uploading: \$uri"

                                    \$ftpReq = [System.Net.FtpWebRequest]::Create(\$uri)
                                    \$ftpReq.Credentials = \$creds
                                    \$ftpReq.Method = [System.Net.WebRequestMethods+Ftp]::UploadFile
                                    \$ftpReq.UseBinary = \$true
                                    \$ftpReq.UsePassive = \$true
                                    \$ftpReq.KeepAlive = \$false
                                    \$ftpReq.Timeout = 15000

                                    \$content = [System.IO.File]::ReadAllBytes(\$_.FullName)
                                    \$ftpStream = \$ftpReq.GetRequestStream()
                                    \$ftpStream.Write(\$content, 0, \$content.Length)
                                    \$ftpStream.Close()
                                    Write-Host "✅ Uploaded: \$relativePath"
                                }
                            }
                            Write-Host "🎉 Upload completed successfully!"
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Deployment failed.'
        }
    }
}
