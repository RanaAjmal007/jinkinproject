pipeline {
    agent any

    options {
        timestamps()
        ansiColor('xterm')
    }

    triggers {
        pollSCM('* * * * *')
    }

    environment {
        PROJECT     = 'JenkinsTest/JenkinsTest/JenkinsTest.csproj'  // relative path inside repo
        CONFIG      = 'Release'
        PUBLISH_DIR = 'build/publish'
        DEPLOY_DIR  = 'C:\\inetpub\\wwwroot\\Jenkinsapp'
        BACKUP_DIR  = 'D:\\IIS_Backup'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                bat 'dir'
            }
        }

        stage('Verify .NET SDK') {
            steps {
                bat 'dotnet --info'
            }
        }

        stage('Restore') {
            steps {
                bat 'dotnet restore "%PROJECT%"'
            }
        }

        stage('Build') {
            steps {
                bat 'dotnet build "%PROJECT%" -c %CONFIG% --no-restore'
            }
        }

        stage('Publish') {
            steps {
                bat 'dotnet publish "%PROJECT%" -c %CONFIG% -o "%PUBLISH_DIR%" --no-build'
            }
        }
        stage('Backup IIS Deployment') {
            steps {
                powershell '''
                    $timestamp = Get-Date -Format "yyyyMMddHHmmss"
                    $backupPath = "${env:BACKUP_DIR}\\Jenkinsapp_$timestamp"
                    $deployPath = "${env:DEPLOY_DIR}"
            
                    if (Test-Path $deployPath) {
                        Write-Host "✅ Starting backup of existing deployment..."
                        New-Item -ItemType Directory -Force -Path $backupPath | Out-Null
                        
                        # Use /XO (Exclude Older) to copy only new or updated files
                        robocopy $deployPath $backupPath /E /XO /V
                        $LastExitCode = $LASTEXITCODE
                        
                        # Check for a real failure (exit codes > 7)
                        if ($LastExitCode -gt 7) {
                            Write-Error "❌ Robocopy backup failed with exit code: $LastExitCode"
                            exit 1
                        }
                        
                        Write-Host "✅ Backup completed with exit code: $LastExitCode"
                    } else {
                        Write-Host "⚠️ No existing deployment found, skipping backup."
                    }
                    # Explicitly exit with a success code
                    exit 0
                '''
            }
        }
          stage('Deploy to IIS') {
                steps {
                    powershell '''
                        Write-Host "🚀 Starting deployment to IIS..."
                        $publishPath = "${env:WORKSPACE}\\build\\publish"
                        $deployPath = "${env:DEPLOY_DIR}"
                        
                        # Robocopy to copy only new or modified files
                        robocopy $publishPath $deployPath /E /XO /FFT /V
                        $LastExitCode = $LASTEXITCODE
                
                        # Check for a successful robocopy operation (codes <= 7)
                        if ($LastExitCode -le 7) {
                            Write-Host "✅ Deployment completed with exit code: $LastExitCode"
                        } else {
                            Write-Error "❌ Robocopy deployment failed with exit code: $LastExitCode"
                            exit 1
                        }
                        
                        # Restart IIS after deployment
                        iisreset /timeout:60
                        Write-Host "✅ IIS has been restarted"
                    '''
                }
}
    
    post {
        success {
            echo "✅ Build & publish complete. Output in: %PUBLISH_DIR%"
        }
        failure {
            echo "❌ Build failed — check stage logs above."
        }
    }
}

