stage('Deploy to IIS via FTP') {
    steps {
        script {
            echo "📦 Uploading build folder recursively to FTP..."

            withCredentials([usernamePassword(credentialsId: 'ftp-credentials', usernameVariable: 'FTP_USER', passwordVariable: 'FTP_PASS')]) {
                powershell '''
                $ErrorActionPreference = "Stop"
                $localPath = "build"
                $ftpServer = "ftp://161.97.130.165"
                $ftpUser = "${env:FTP_USER}"
                $ftpPass = "${env:FTP_PASS}"

                Write-Host "🔍 Testing FTP connection to $ftpServer ..."
                try {
                    $request = [System.Net.WebRequest]::Create("$ftpServer")
                    $request.Method = [System.Net.WebRequestMethods+Ftp]::ListDirectory
                    $request.Credentials = New-Object System.Net.NetworkCredential($ftpUser, $ftpPass)
                    $response = $request.GetResponse()
                    $response.Close()
                    Write-Host "✅ FTP connection OK!"
                } catch {
                    Write-Host "❌ FTP connection failed!"
                    Write-Host $_.Exception.Message
                    exit 1
                }

                Function Upload-FtpFile($source, $target, $credentials) {
                    try {
                        $uri = New-Object System.Uri($target)
                        $ftpRequest = [System.Net.FtpWebRequest]::Create($uri)
                        $ftpRequest.Credentials = $credentials
                        $ftpRequest.Method = [System.Net.WebRequestMethods+Ftp]::UploadFile
                        $ftpRequest.UseBinary = $true
                        $ftpRequest.UsePassive = $true
                        $ftpRequest.Timeout = 10000

                        $content = [System.IO.File]::ReadAllBytes($source)
                        $ftpStream = $ftpRequest.GetRequestStream()
                        $ftpStream.Write($content, 0, $content.Length)
                        $ftpStream.Close()
                        Write-Host "📤 Uploaded: $($source -replace '^build\\\\','')"
                    } catch {
                        Write-Host "⚠️ Failed to upload: $($source -replace '^build\\\\','')"
                        Write-Host $_.Exception.Message
                    }
                }

                $credentials = New-Object System.Net.NetworkCredential($ftpUser, $ftpPass)
                $files = Get-ChildItem -Recurse -File $localPath

                foreach ($file in $files) {
                    $relativePath = $file.FullName.Substring($localPath.Length + 1).Replace("\\", "/")
                    $remoteFile = "$ftpServer/$relativePath"
                    Upload-FtpFile $file.FullName $remoteFile $credentials
                }
                '''
            }
        }
    }
}
