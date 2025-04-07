pipeline {
    agent { label 'universal-agent' }

    tools {
        maven 'M3'
    }

    stages {
        stage('Detect Changed Service & Build') {
            steps {
                checkout scm
                script {
                    def servicePaths = [
                        'spring-petclinic-admin-server',
                        'spring-petclinic-api-gateway',
                        'spring-petclinic-config-server',
                        'spring-petclinic-customers-service',
                        'spring-petclinic-discovery-server',
                        'spring-petclinic-genai-service',
                        'spring-petclinic-vets-service',
                        'spring-petclinic-visits-service'
                    ]

                    def changedFiles = []
                    if (env.CHANGE_ID) {
                        changedFiles = sh(script: "git diff --name-only origin/main", returnStdout: true).trim().split("\n")
                    } else {
                        changedFiles = sh(script: "git diff --name-only HEAD~1", returnStdout: true).trim().split("\n")
                    }

                    def detectedService = servicePaths.find { service ->
                        changedFiles.any { it.startsWith(service + "/") }
                    }

                    if (detectedService) {
                        echo "🔍 Detected change in: ${detectedService}"
                        def mvnCmd = isUnix() ? './mvnw' : 'mvnw.cmd'
                        if (isUnix()) {
                            sh "${mvnCmd} -pl ${detectedService} -am clean verify"
                        } else {
                            bat "${mvnCmd} -pl ${detectedService} -am clean verify"
                        }

                        junit "${detectedService}/target/surefire-reports/*.xml"
                        recordCoverage(
                            tools: [[parser: 'JACOCO', pattern: "${detectedService}/target/site/jacoco/jacoco.xml"]],
                            qualityGates: [
                                [threshold: 70.0, metric: 'LINE', baseline: 'PROJECT', unstable: false],
                                [threshold: 70.0, metric: 'BRANCH', baseline: 'PROJECT', unstable: false]
                            ]
                        )
                    } else {
                        echo "🟡 Không phát hiện thay đổi liên quan đến service nào – Skip build."
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build thành công!'
        }
        failure {
            echo '❌ Build thất bại!'
        }
    }
}
