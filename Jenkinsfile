pipeline {
    agent { label 'universal-agent' }

    tools {
        maven 'M3'
    }

    parameters {
        string(name: 'DEPLOY_BRANCH', defaultValue: 'main', description: 'Tên nhánh bạn muốn deploy')
    }

    stages {
        stage('📥 Clone repo') {
            steps {
                git branch: "${params.DEPLOY_BRANCH}", 
                    url: 'https://github.com/NPT0116/thanh-microservices-spring.git', 
                    credentialsId: 'github-thanh-token'
            }
        }

        stage('🔨 Build vets-service') {
            steps {
                script {
                    def mvnCmd = isUnix() ? './mvnw' : 'mvnw.cmd'
                    def cmd = "${mvnCmd} -pl spring-petclinic-vets-service -am clean install -DskipTests"
                    isUnix() ? sh(cmd) : bat(cmd)
                }
            }
        }

        stage('🐳 Docker Build & Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-login', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        def loginCmd = isUnix() 
                            ? "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin" 
                            : "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"

                        def buildCmd = isUnix() 
                            ? "./mvnw -pl spring-petclinic-vets-service spring-boot:build-image -DskipTests -Dspring-boot.build-image.imageName=npt1601/spring-petclinic-vets-service:latest"
                            : "mvnw.cmd -pl spring-petclinic-vets-service spring-boot:build-image -DskipTests -Dspring-boot.build-image.imageName=npt1601/spring-petclinic-vets-service:latest"

                        def pushCmd = isUnix() 
                            ? "docker push npt1601/spring-petclinic-vets-service:latest" 
                            : "docker push npt1601/spring-petclinic-vets-service:latest"

                        isUnix() ? sh(loginCmd) : bat(loginCmd)
                        isUnix() ? sh(buildCmd) : bat(buildCmd)
                        isUnix() ? sh(pushCmd) : bat(pushCmd)
                    }
                }
            }
        }

        stage('🚀 Deploy to Minikube') {
            steps {
                script {
                    def kubeConfigPath = isUnix() ? '/Users/npt/.kube/config' : 'C:\\Users\\npt\\.kube\\config'
                    def applyCmd = isUnix()
                        ? "export KUBECONFIG=${kubeConfigPath} && kubectl apply -f k8s/vets-deployment.yaml"
                        : "set KUBECONFIG=${kubeConfigPath} && kubectl apply -f k8s\\vets-deployment.yaml"

                    isUnix() ? sh(applyCmd) : bat(applyCmd)
                }
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD cho vets-service thành công!'
        }
        failure {
            echo '❌ CI/CD thất bại!'
        }
    }
}
