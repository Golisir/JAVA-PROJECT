pipeline {
    agent any

    environment {
        APP_DIR = "app"
        GIT_CREDENTIALS_ID = "git-id"
        SCANNER_HOME= "sonar-scanner"
        AWS_REGION   = "us-east-1"  //region
        AWS_ACCOUNT  = "559050204886" //AWS account ID
        ECR_REPO     = "sankarrepository"  //repo name
        IMAGE_TAG = "${BUILD_NUMBER}"       // or use ${env.BUILD_NUMBER}
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

             stage('Build Docker Image') {
            steps {
                script {
                    dir("${APP_DIR}") {
                    docker.build("${env.ECR_REPO}:${env.IMAGE_TAG}")
                }
            }
        }
    }

        stage('Push to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    script {
                        def ecrRepo = "${env.AWS_ACCOUNT}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.ECR_REPO}:${env.IMAGE_TAG}"

                        sh """
                        aws ecr get-login-password --region ${env.AWS_REGION} \
                          | docker login --username AWS --password-stdin ${env.AWS_ACCOUNT}.dkr.ecr.${env.AWS_REGION}.amazonaws.com

                        docker tag ${env.ECR_REPO}:${env.IMAGE_TAG} ${ecrRepo}
                        docker push ${ecrRepo}
                        """
                    }
                }
            }
        }
    }
}
