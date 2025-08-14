pipeline {
    agent any

    environment {
        APP_DIR = "app"
        GIT_CREDENTIALS_ID = "git-id"
        SCANNER_HOME= "sonar-scanner"
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
                git(
                    branch: "${env.ACTUAL_BRANCH}",
                    url: 'https://github.com/Golisir/JAVA-PROJECT.git',
                    credentialsId: "${GIT_CREDENTIALS_ID}"
                )
            }
        }

        stage('SonarQube Analysis') {
            when {
                expression { return ['dev', 'uat', 'main'].contains(env.ACTUAL_BRANCH) }
            }
            steps {
                dir("${APP_DIR}") {
                    withSonarQubeEnv('My SonarQube') {
                        sh 'mvn sonar:sonar -Dsonar.projectKey=java-project -Dsonar.host.url=http://13.222.104.69:9000/'
                    }
                }
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
