pipeline {
    agent { label 'universal-agent' }

    tools {
        maven 'M3'
    }

    parameters {
        string(name: 'DEPLOY_BRANCH', defaultValue: 'main', description: 'Tên nhánh bạn muốn deploy')
    }

    stages {
        stage('Clone code từ GitHub') {
            steps {
                git branch: "${params.DEPLOY_BRANCH}", url: 'https://github.com/NPT0116/thanh-microservices-spring.git', credentialsId: 'github-thanh-token'
            }
        }

        stage('Build toàn bộ services') {
            steps {
                script {
                    def mvnCmd = isUnix() ? './mvnw' : 'mvnw.cmd'
                    if (isUnix()) {
                        sh "${mvnCmd} clean install -DskipTests"
                    } else {
                        bat "${mvnCmd} clean install -DskipTests"
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    def dockerLogin = isUnix() ? 'docker login -u $DOCKER_USER -p $DOCKER_PASS' : 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
                    def dockerBuild = isUnix() ? './mvnw spring-boot:build-image -DskipTests' : 'mvnw.cmd spring-boot:build-image -DskipTests'

                    withCredentials([usernamePassword(credentialsId: 'dockerhub-login', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        if (isUnix()) {
                            sh dockerLogin
                            sh dockerBuild
                        } else {
                            bat dockerLogin
                            bat dockerBuild
                        }
                    }
                }
            }
        }

        stage('Deploy lên Minikube') {
            steps {
                script {
                    def kubeConfigPath = isUnix() ? '/Users/npt/.kube/config' : 'C:\\Users\\npt\\.kube\\config'
                    echo "📦 Deploying using KUBECONFIG: ${kubeConfigPath}"

                    if (isUnix()) {
                        sh "export KUBECONFIG=${kubeConfigPath} && kubectl apply -f k8s/"
                    } else {
                        bat "set KUBECONFIG=${kubeConfigPath} && kubectl apply -f k8s\\"
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD hoàn tất thành công!'
        }
        failure {
            echo '❌ CI/CD thất bại!'
        }
    }
}
