
pipeline {
    agent any

    parameters {
        choice(
            name: 'BLOCKING_THRESHOLD',
            choices: ['Medium', 'High'],
            description: 'Minimum risk level that blocks the build.'
        )
    }

    environment {
        DOCKER = 'C:\\Users\\ombad\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        ZAP_IMAGE = 'ghcr.io/zaproxy/zaproxy:stable'
        TARGET = 'http://host.docker.internal:3000'
    }

    stages {
        stage('Check Docker') {
            steps {
                bat '"%DOCKER%" ps'
            }
        }

        stage('Run OWASP ZAP Baseline Scan') {
            steps {
                script {
                    int zapExitCode = bat(
                        returnStatus: true,
                        script: '''
                            "%DOCKER%" run --rm ^
                              -v "%WORKSPACE%:/zap/wrk/:rw" ^
                              "%ZAP_IMAGE%" ^
                              zap-baseline.py ^
                              -t "%TARGET%" ^
                              -r zap-report.html ^
                              -J zap-report.json
                        '''
                    )

                    echo "OWASP ZAP exit code: ${zapExitCode}"

                    if (zapExitCode < 0 || zapExitCode > 2) {
                        error("ZAP execution failed with exit code ${zapExitCode}")
                    }
                }
            }
        }

        stage('Archive Security Reports') {
            steps {
                archiveArtifacts(
                    artifacts: 'zap-report.html,zap-report.json',
                    allowEmptyArchive: false
                )
            }
        }

        stage('Security Gate') {
            steps {
                script {
                    def report = readJSON(file: 'zap-report.json')
                    def alerts = (report.site ?: []).collectMany {
                        it.alerts ?: []
                    }

                    def blockingAlerts = alerts.findAll { alert ->
                        String risk = (alert.riskdesc ?: '')
                            .toLowerCase()
                            .trim()

                        if (params.BLOCKING_THRESHOLD == 'High') {
                            return risk.startsWith('high')
                        }

                        return risk.startsWith('high') ||
                               risk.startsWith('medium')
                    }

                    echo "Selected threshold: ${params.BLOCKING_THRESHOLD}"
                    echo "Total alert entries: ${alerts.size()}"
                    echo "Blocking alert entries: ${blockingAlerts.size()}"

                    blockingAlerts.each { alert ->
                        echo "BLOCKED: ${alert.riskdesc} - ${alert.name}"
                    }

                    if (!blockingAlerts.isEmpty()) {
                        error(
                            "Security gate failed: alerts meet the ${params.BLOCKING_THRESHOLD} threshold."
                        )
                    }

                    echo 'Security gate passed.'
                }
            }
        }
    }

    post {
        always {
            echo 'DevSecOps security pipeline finished.'
        }
    }
}
