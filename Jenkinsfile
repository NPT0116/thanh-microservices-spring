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
        string(name: 'VETS_BRANCH',       defaultValue: 'main', description: 'Branch của vets-service')
        string(name: 'CUSTOMERS_BRANCH',  defaultValue: 'main', description: 'Branch của customers-service')
        string(name: 'VISITS_BRANCH',     defaultValue: 'main', description: 'Branch của visits-service')
        string(name: 'GATEWAY_BRANCH',    defaultValue: 'main', description: 'Branch của api-gateway')
        string(name: 'CONFIG_BRANCH',     defaultValue: 'main', description: 'Branch của config-server')
        string(name: 'DISCOVERY_BRANCH',  defaultValue: 'main', description: 'Branch của discovery-server')
        string(name: 'ADMIN_BRANCH',      defaultValue: 'main', description: 'Branch của admin-server')
    }

    stages {
        stage('🚀 Deploy các service') {
            steps {
                script {
                    def home = isUnix() ? env.HOME : env.USERPROFILE
                    def kubeConfigPath = "${home}${isUnix() ? '/.kube/config' : '\\.kube\\config'}"

                    def services = [
                        [name: 'vets',       image: 'npt1601/vets-service',      branch: params.VETS_BRANCH],
                        [name: 'customers',  image: 'npt1601/customers-service', branch: params.CUSTOMERS_BRANCH],
                        [name: 'visits',     image: 'npt1601/visits-service',    branch: params.VISITS_BRANCH],
                        [name: 'gateway',    image: 'npt1601/api-gateway',       branch: params.GATEWAY_BRANCH],
                        [name: 'config',     image: 'npt1601/config-server',     branch: params.CONFIG_BRANCH],
                        [name: 'discovery',  image: 'npt1601/discovery-server',  branch: params.DISCOVERY_BRANCH],
                        [name: 'admin',      image: 'npt1601/admin-server',      branch: params.ADMIN_BRANCH]
                    ]

                    services.each { svc ->
                        def tag = (svc.branch == 'main') 
                            ? 'latest' 
                            : getCommitId(svc.branch)

                        def fullImage = "${svc.image}:${tag}"
                        echo "⚙️ Triển khai ${svc.name}-deployment với image: ${fullImage}"

                        def deployCmd = isUnix()
                            ? "export KUBECONFIG=${kubeConfigPath} && kubectl set image deployment/${svc.name}-deployment ${svc.name}=${fullImage}"
                            : "set KUBECONFIG=${kubeConfigPath} && kubectl set image deployment/${svc.name}-deployment ${svc.name}=${fullImage}"

                        isUnix() ? sh(deployCmd) : bat(deployCmd)
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Đã triển khai thành công các service!'
        }
        failure {
            echo '❌ Triển khai thất bại!'
        }
    }
}

def getCommitId(String branchName) {
    if (isUnix()) {
        return sh(script: "git fetch origin ${branchName} && git rev-parse --short origin/${branchName}", returnStdout: true).trim()
    } else {
        def result = bat(script: "git fetch origin ${branchName} && git rev-parse --short origin/${branchName}", returnStdout: true).trim()
        return result.readLines().last().trim()
    }
}
