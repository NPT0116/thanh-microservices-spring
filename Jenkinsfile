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

    parameters {
        choice(name: 'SERVICE_NAME', choices: [
            'vets-service',
            'customers-service',
            'visits-service',
            'api-gateway',
            'config-server',
            'discovery-server',
            'admin-server',
            'genai-service',
            'tracing-server'
        ], description: 'Chọn service cần build')

        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Tên branch bạn muốn build')
    }

    stages {
        stage('📥 Checkout') {
            steps {
                git branch: "${params.BRANCH_NAME}",
                    url: 'https://github.com/NPT0116/thanh-microservices-spring.git',
                    credentialsId: 'github-thanh-token'
            }
        }

        stage('🔨 Build & Push image') {
            steps {
                script {
                    def commitId = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    def imageName = "npt1601/${params.SERVICE_NAME}:${commitId}"

                    echo "📦 Building image ${imageName}"

                    withCredentials([usernamePassword(credentialsId: 'dockerhub-login', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        def loginCmd = isUnix()
                            ? "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                            : "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                        def buildCmd = isUnix()
                            ? "./mvnw -pl spring-petclinic-${params.SERVICE_NAME} spring-boot:build-image -DskipTests -Dspring-boot.build-image.imageName=${imageName}"
                            : "mvnw.cmd -pl spring-petclinic-${params.SERVICE_NAME} spring-boot:build-image -DskipTests -Dspring-boot.build-image.imageName=${imageName}"
                        def pushCmd = "docker push ${imageName}"

                        isUnix() ? sh(loginCmd) : bat(loginCmd)
                        isUnix() ? sh(buildCmd) : bat(buildCmd)
                        isUnix() ? sh(pushCmd) : bat(pushCmd)
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build & Push thành công!'
        }
        failure {
            echo '❌ Build thất bại!'
        }
    }
}
