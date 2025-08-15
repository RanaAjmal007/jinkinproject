pipeline {
    agent any

    options {
        timestamps()
        ansiColor('xterm')
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        PROJECT     = 'JenkinsTest/JenkinsTest.csproj'  // relative path inside repo
        CONFIG      = 'Release'
        PUBLISH_DIR = 'build/publish'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
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
