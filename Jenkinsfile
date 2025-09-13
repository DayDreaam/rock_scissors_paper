pipeline {
    agent any

    environment {
        AWS_REGION = "ap-northeast-2"  // AWS 리전 설정
        AWS_ACCOUNT_ID = "149536459102"  // AWS 계정 ID (ECR 저장소 소유자)
        ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/my-jenkins-image" // AWS ECR 저장소 URL
        IMAGE_TAG = "latest"  // 기본 태그 설정
    }

    stages {
        stage('Ensure ECR Exists') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials'
                ]]) {
                    // ECR 저장소가 존재하는지 확인하고 없으면 생성
                    sh """
                    aws ecr describe-repositories --repository-names my-jenkins-image --region ${AWS_REGION} || \
                    aws ecr create-repository --repository-name my-jenkins-image --region ${AWS_REGION}
                    """
                }
            }
        }

        stage('Checkout') {
            steps {
                // GitHub 저장소에서 코드 가져오기
                git branch: 'main',
                credentialsId: '18a8ec02-e585-40e0-8451-8a000bd0ff6b',
                url: 'git@github.com:DayDreaam/rock_scissors_paper.git'
            }
        }

        stage('Build JAR') {
            steps {
                sh "chmod +x ./gradlew"
                // gradle 빌드하기
                sh "./gradlew build"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Docker 이미지 빌드 (태그 포함)
                    sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
                }
            }
        }

        stage('AWS ECR Login') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials']]) {
                        script {
                            // AWS ECR 로그인 수행
                            sh """
                            aws ecr get-login-password --region ${AWS_REGION} | \
                            docker login --username AWS --password-stdin ${ECR_REPO}
                            """
                        }
                    }
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    // Docker 이미지를 ECR에 Push
                    sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
                }
            }
        }
    }
}