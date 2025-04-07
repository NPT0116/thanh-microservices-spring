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
        string(name: 'VETS_BRANCH', defaultValue: 'main', description: 'Branch của vets-service')
        string(name: 'CUSTOMERS_BRANCH', defaultValue: 'main', description: 'Branch của customers-service')
        string(name: 'VISITS_BRANCH', defaultValue: 'main', description: 'Branch của visits-service')
        string(name: 'GATEWAY_BRANCH', defaultValue: 'main', description: 'Branch của api-gateway')
        string(name: 'CONFIG_BRANCH', defaultValue: 'main', description: 'Branch của config-server')
        string(name: 'DISCOVERY_BRANCH', defaultValue: 'main', description: 'Branch của discovery-server')
        string(name: 'ADMIN_BRANCH', defaultValue: 'main', description: 'Branch của admin-server')
    }

    stages {

        stage('📥 Checkout') {
            steps {
                git branch: "${params.VETS_BRANCH}",
                    url: 'https://github.com/NPT0116/thanh-microservices-spring.git',
                    credentialsId: 'github-thanh-token'
            }
        }

        stage('🔨 Build & Push Images') {
            steps {
                script {
                    def services = [
                        [name: 'vets-service',       branch: params.VETS_BRANCH],
                        [name: 'customers-service',  branch: params.CUSTOMERS_BRANCH],
                        [name: 'visits-service',     branch: params.VISITS_BRANCH],
                        [name: 'api-gateway',        branch: params.GATEWAY_BRANCH],
                        [name: 'config-server',      branch: params.CONFIG_BRANCH],
                        [name: 'discovery-server',   branch: params.DISCOVERY_BRANCH],
                        [name: 'admin-server',       branch: params.ADMIN_BRANCH]
                    ]

                    withCredentials([usernamePassword(credentialsId: 'dockerhub-login', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        def loginCmd = isUnix()
                            ? "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                            : "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                        isUnix() ? sh(loginCmd) : bat(loginCmd)

                        services.each { svc ->
                            echo "🔨 Building ${svc.name} from branch ${svc.branch}"
                            def commitId = sh(script: "git rev-parse --short origin/${svc.branch}", returnStdout: true).trim()

                            def buildCmd = isUnix()
                                ? "./mvnw -pl spring-petclinic-${svc.name} spring-boot:build-image -DskipTests -Dspring-boot.build-image.imageName=npt1601/${svc.name}:${commitId}"
                                : "mvnw.cmd -pl spring-petclinic-${svc.name} spring-boot:build-image -DskipTests -Dspring-boot.build-image.imageName=npt1601/${svc.name}:${commitId}"
                            def pushCmd = "docker push npt1601/${svc.name}:${commitId}"

                            isUnix() ? sh(buildCmd) : bat(buildCmd)
                            isUnix() ? sh(pushCmd) : bat(pushCmd)
                        }
                    }
                }
            }
        }

        stage('🚀 Deploy to Minikube') {
            steps {
                script {
                    def home = isUnix() ? env.HOME : env.USERPROFILE
                    def kubeConfigPath = "${home}${isUnix() ? '/.kube/config' : '\\.kube\\config'}"

                    def services = [
                        [name: 'vets',       image: "npt1601/vets-service",       tag: getTag(params.VETS_BRANCH)],
                        [name: 'customers',  image: "npt1601/customers-service",  tag: getTag(params.CUSTOMERS_BRANCH)],
                        [name: 'visits',     image: "npt1601/visits-service",     tag: getTag(params.VISITS_BRANCH)],
                        [name: 'gateway',    image: "npt1601/api-gateway",        tag: getTag(params.GATEWAY_BRANCH)],
                        [name: 'config',     image: "npt1601/config-server",      tag: getTag(params.CONFIG_BRANCH)],
                        [name: 'discovery',  image: "npt1601/discovery-server",   tag: getTag(params.DISCOVERY_BRANCH)],
                        [name: 'admin',      image: "npt1601/admin-server",       tag: getTag(params.ADMIN_BRANCH)]
                    ]

                    services.each { svc ->
                        def applyYaml = isUnix()
                            ? "export KUBECONFIG=${kubeConfigPath} && kubectl set image deployment/${svc.name}-deployment ${svc.name}=${svc.image}:${svc.tag}"
                            : "set KUBECONFIG=${kubeConfigPath} && kubectl set image deployment/${svc.name}-deployment ${svc.name}=${svc.image}:${svc.tag}"
                        echo "🚀 Updating ${svc.name}-deployment to image: ${svc.image}:${svc.tag}"
                        isUnix() ? sh(applyYaml) : bat(applyYaml)
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD cho toàn bộ hệ thống thành công!'
        }
        failure {
            echo '❌ CI/CD thất bại!'
        }
    }
}

def getTag(branch) {
    return branch == 'main' ? 'latest' : sh(script: "git rev-parse --short origin/${branch}", returnStdout: true).trim()
}
