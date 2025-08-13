pipeline {
    agent any

    environment {
        APP_DIR = "app" // define APP_DIR if used later
         GIT_CREDENTIALS_ID = "git-id" // uncomment if you use credentials
    }

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
                    url: 'https://github.com/Golisir/JAVA-PROJECT.git'
                 credentialsId: "${GIT_CREDENTIALS_ID}" // uncomment if needed
            }
        }

        stage('Maven Build') {
            when {
                expression { return ['dev', 'uat', 'main'].contains(env.ACTUAL_BRANCH) }
            }
            steps {
                dir("${APP_DIR}") {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }
    }
}
