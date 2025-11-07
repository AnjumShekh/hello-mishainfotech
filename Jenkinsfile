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

                    withCredentials([usernamePassword(credentialsId: 'ftp-credentials', usernameVariable: 'FTP_USER', passwordVariable: 'FTP_PASS')]) {
                        powershell '''
                            Write-Host "🚀 Starting recursive FTP upload..."

                            $ftp = "ftp://161.97.130.165/"
                            $webClient = New-Object System.Net.WebClient
                            $webClient.Credentials = New-Object System.Net.NetworkCredential($env:FTP_USER, $env:FTP_PASS)

                            $buildPath = Resolve-Path "build"

                            Get-ChildItem -Recurse -Path $buildPath | ForEach-Object {
                                if (-not $_.PSIsContainer) {
                                    $relativePath = $_.FullName.Substring($buildPath.Path.Length + 1).Replace("\\", "/")
                                    $uri = [System.Uri]::EscapeUriString($ftp + $relativePath)

                                    Write-Host "📤 Uploading: $relativePath"
                                    try {
                                        $webClient.UploadFile($uri, "STOR", $_.FullName)
                                    } catch {
                                        Write-Host "⚠️ Failed to upload: $relativePath"
                                    }
                                }
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
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}
