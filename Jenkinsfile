pipeline {
    agent any

    stages {
        stage('Determine Branch') {
            steps {
                script {
                    env.ACTUAL_BRANCH = env.BRANCH_NAME ?: env.GIT_BRANCH?.replaceFirst(/^origin\//, '') ?: 'dev'
                    echo "🔀 Detected branch: ${env.ACTUAL_BRANCH}"
                }
            }
        }

    stage('Clone Repository') {
      steps {
        git branch: "${env.ACTUAL_BRANCH}",
             url: 'https://github.com/krishnasravi/kubernetes-demo.git',
             credentialsId: "${GIT_CREDENTIALS_ID}"
      }
    }


