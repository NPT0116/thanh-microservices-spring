pipeline {
    agent { label 'universal-agent' }

    tools {
        jdk 'jdk21'
        maven 'M3'
    }

    environment {
        JAVA_HOME = "${tool 'jdk21'}"
        PATH = "${env.JAVA_HOME}/bin${isUnix() ? ':' : ';'}${env.PATH}"
    }

    stages {
        stage('📥 Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME ?: 'main'}",
                    url: 'https://github.com/NPT0116/thanh-microservices-spring.git'
            }
        }

        stage('📦 Build & Push tất cả service') {
            steps {
                script {
                    def commitId = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    echo "🧬 Commit ID hiện tại: ${commitId}"

                    def services = [
                        'vets-service',
                        'customers-service',
                        'visits-service',
                        'api-gateway',
                        'config-server',
                        'discovery-server',
                        'admin-server'
                    ]

                    withCredentials([usernamePassword(credentialsId: 'dockerhub-login', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        def loginCmd = isUnix()
                            ? "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                            : "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"

                        isUnix() ? sh(loginCmd) : bat(loginCmd)

                        services.each { svc ->
                            def image = "npt1601/${svc}:${commitId}"
                            def buildCmd = isUnix()
                                ? "./mvnw -pl spring-petclinic-${svc} spring-boot:build-image -DskipTests -Dspring-boot.build-image.imageName=${image}"
                                : "mvnw.cmd -pl spring-petclinic-${svc} spring-boot:build-image -DskipTests -Dspring-boot.build-image.imageName=${image}"

                            def pushCmd = "docker push ${image}"

                            echo "🚧 Đang build image cho ${svc} → ${image}"
                            isUnix() ? sh(buildCmd) : bat(buildCmd)
                            echo "📤 Push image lên DockerHub: ${image}"
                            isUnix() ? sh(pushCmd) : bat(pushCmd)
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ CI hoàn tất! Docker images đã được đẩy lên Docker Hub với tag commit ID."
        }
        failure {
            echo "❌ CI thất bại!"
        }
    }
}
