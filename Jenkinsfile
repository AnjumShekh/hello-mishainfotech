stage('Deploy to IIS via FTP') {
  steps {
    script {
      echo "📦 Uploading build folder recursively to FTP root..."
      withCredentials([usernamePassword(credentialsId: 'ftp-credentials', usernameVariable: 'FTP_USER', passwordVariable: 'FTP_PASS')]) {
        powershell '''
          $sourcePath = "build"
          $ftpServer = "ftp://161.97.130.165"   # Uploading to root
          $ftpUser = $env:FTP_USER
          $ftpPass = $env:FTP_PASS

          function New-FtpDirectory {
            param($ftpUrl, $credentials)
            try {
              $req = [System.Net.FtpWebRequest]::Create($ftpUrl)
              $req.Credentials = $credentials
              $req.Method = [System.Net.WebRequestMethods+Ftp]::MakeDirectory
              $req.GetResponse().Close()
              Write-Host "📁 Created folder: $ftpUrl"
            } catch {
              # Ignore 'folder already exists' errors
            }
          }

          $creds = New-Object System.Net.NetworkCredential($ftpUser, $ftpPass)

          Get-ChildItem -Path $sourcePath -Recurse | ForEach-Object {
            if (-not $_.PSIsContainer) {
              $relativePath = $_.FullName.Substring($sourcePath.Length + 1).Replace("\\", "/")
              $remoteDir = [System.IO.Path]::GetDirectoryName($relativePath).Replace("\\", "/")

              # Create directory on FTP if it doesn’t exist
              if ($remoteDir -ne "") {
                $ftpDirUrl = "$ftpServer/$remoteDir"
                New-FtpDirectory -ftpUrl $ftpDirUrl -credentials $creds
              }

              # Upload file
              $uri = "$ftpServer/$relativePath"
              Write-Host "⬆️ Uploading: $uri"
              $ftpReq = [System.Net.FtpWebRequest]::Create($uri)
              $ftpReq.Credentials = $creds
              $ftpReq.Method = [System.Net.WebRequestMethods+Ftp]::UploadFile
              $ftpReq.UseBinary = $true
              $ftpReq.UsePassive = $true
              $ftpReq.KeepAlive = $false
              $ftpReq.Timeout = 10000
              $content = [System.IO.File]::ReadAllBytes($_.FullName)
              $ftpStream = $ftpReq.GetRequestStream()
              $ftpStream.Write($content, 0, $content.Length)
              $ftpStream.Close()
              Write-Host "✅ Uploaded: $relativePath"
            }
          }
        '''
      }
    }
  }
}
