pipeline {
    agent any

    environment {
        FTP_SERVER = '161.97.130.165'
        FTP_PORT = '21'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/AnjumShekh/hello-mishainfotech.git'
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

                    withCredentials([usernamePassword(credentialsId: 'ftp-credentials', usernameVariable: 'FTP_USER', passwordVariable: 'FTP_PASS')]) {
                        powershell '''
                        $localPath = "build"
                        $ftpServer = "ftp://161.97.130.165:21"
                        $ftpUser = "${env:FTP_USER}"
                        $ftpPass = "${env:FTP_PASS}"

                        Function Create-FtpDirectory($url, $credentials) {
                            try {
                                $request = [System.Net.WebRequest]::Create($url)
                                $request.Method = [System.Net.WebRequestMethods+Ftp]::MakeDirectory
                                $request.Credentials = $credentials
                                $response = $request.GetResponse()
                                $response.Close()
                                Write-Host "📁 Created directory: $url"
                            } catch {
                                # Ignore if already exists
                            }
                        }

                        Function Upload-FtpFile($source, $target, $credentials) {
                            try {
                                $uri = New-Object System.Uri($target)
                                $ftpRequest = [System.Net.FtpWebRequest]::Create($uri)
                                $ftpRequest.Credentials = $credentials
                                $ftpRequest.Method = [System.Net.WebRequestMethods+Ftp]::UploadFile
                                $ftpRequest.UseBinary = $true
                                $ftpRequest.UsePassive = $true

                                $content = [System.IO.File]::ReadAllBytes($source)
                                $ftpStream = $ftpRequest.GetRequestStream()
                                $ftpStream.Write($content, 0, $content.Length)
                                $ftpStream.Close()
                                Write-Host "📤 Uploaded: $($source -replace '^build\\\\','')"
                            } catch {
                                Write-Host "⚠️ Failed to upload: $($source -replace '^build\\\\','')"
                            }
                        }

                        $credentials = New-Object System.Net.NetworkCredential($ftpUser, $ftpPass)
                        $files = Get-ChildItem -Recurse -File $localPath

                        foreach ($file in $files) {
                            $relativePath = $file.FullName.Substring($localPath.Length + 1).Replace("\\", "/")
                            $remoteFile = "$ftpServer/$relativePath"
                            $remoteDir = [System.IO.Path]::GetDirectoryName($remoteFile).Replace("\\", "/")

                            # Create directory if it doesn't exist
                            Create-FtpDirectory $remoteDir $credentials

                            # Upload file
                            Upload-FtpFile $file.FullName $remoteFile $credentials
                        }

                        Write-Host "✅ FTP upload completed successfully!"
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
            echo "🌐 Visit your site at: http://161.97.130.165:8072/"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}
