pipeline {
    agent { label 'universal-agent' }

    tools {
        maven 'M3'
    }

    parameters {
        string(name: 'DEPLOY_BRANCH', defaultValue: 'main', description: 'Tên nhánh bạn muốn deploy')
    }

    stages {
        stage('Clone repo') {
            steps {
                git branch: "${params.DEPLOY_BRANCH}", 
                    url: 'https://github.com/NPT0116/thanh-microservices-spring.git', 
                    credentialsId: 'github-thanh-token'
            }
        }

        stage('Build vets-service') {
            steps {
                script {
                    def mvnCmd = isUnix() ? './mvnw' : 'mvnw.cmd'
                    if (isUnix()) {
                        sh "${mvnCmd} -pl spring-petclinic-vets-service -am clean install -DskipTests"
                    } else {
                        bat "${mvnCmd} -pl spring-petclinic-vets-service -am clean install -DskipTests"
                    }
                }
            }
        }

        stage('Docker Build & Push vets-service') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-login', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        def loginCmd = isUnix() ? "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin" : "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                        def buildCmd = isUnix() 
                            ? "./mvnw -pl spring-petclinic-vets-service spring-boot:build-image -DskipTests -Dspring-boot.build-image.imageName=thanh1612004/spring-petclinic-vets-service:latest"
                            : "mvnw.cmd -pl spring-petclinic-vets-service spring-boot:build-image -DskipTests -Dspring-boot.build-image.imageName=thanh1612004/spring-petclinic-vets-service:latest"
                        
                        if (isUnix()) {
                            sh loginCmd
                            sh buildCmd
                            sh "docker push thanh1612004/spring-petclinic-vets-service:latest"
                        } else {
                            bat loginCmd
                            bat buildCmd
                            bat "docker push thanh1612004/spring-petclinic-vets-service:latest"
                        }
                    }
                }
            }
        }

        stage('Deploy vets-service lên Minikube') {
            steps {
                script {
                    def kubeConfigPath = isUnix() ? '/Users/npt/.kube/config' : 'C:\\Users\\npt\\.kube\\config'
                    if (isUnix()) {
                        sh "export KUBECONFIG=${kubeConfigPath} && kubectl apply -f k8s/vets-deployment.yaml"
                    } else {
                        bat "set KUBECONFIG=${kubeConfigPath} && kubectl apply -f k8s\\vets-deployment.yaml"
                    }
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
