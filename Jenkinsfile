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
                        [name: 'vets-service',       branch: params.VETS_BRANCH],
                        [name: 'customers-service',  branch: params.CUSTOMERS_BRANCH],
                        [name: 'visits-service',     branch: params.VISITS_BRANCH],
                        [name: 'api-gateway',        branch: params.GATEWAY_BRANCH],
                        [name: 'config-server',      branch: params.CONFIG_BRANCH],
                        [name: 'discovery-server',   branch: params.DISCOVERY_BRANCH],
                        [name: 'admin-server',       branch: params.ADMIN_BRANCH]
                    ]

                    services.each { svc ->
                        def tag = (svc.branch == 'main') 
                            ? 'latest' 
                            : getCommitId(svc.branch)

                        def fullImage = "npt1601/${svc.name}:${tag}"
                        def deploymentName = "${svc.name}-deployment"
                        def containerName = svc.name

                        echo "⚙️ Triển khai ${deploymentName} với image: ${fullImage}"

                        def deployCmd = isUnix()
                            ? "export KUBECONFIG=${kubeConfigPath} && kubectl set image deployment/${deploymentName} ${containerName}=${fullImage}"
                            : "set KUBECONFIG=${kubeConfigPath} && kubectl set image deployment/${deploymentName} ${containerName}=${fullImage}"

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
