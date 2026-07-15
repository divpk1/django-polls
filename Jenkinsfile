pipeline {
    agent any

    parameters {
        string(name: 'PR_NUMBER', defaultValue: '0', description: 'GitHub PR Number')
        string(name: 'PR_BRANCH', defaultValue: 'master', description: 'Source Branch')
        string(name: 'PR_BASE',   defaultValue: 'master', description: 'Target Base Branch')
    }

    environment {
        SONAR_SERVER_NAME  = 'SonarQube-Public-Server'
        SONAR_SCANNER_HOME = tool 'SonarQubeScanner'
    }

    stages {
        stage('Prepare Branch') {
            steps {
                script {
                    echo "Ensuring workspace is on branch: ${params.PR_BRANCH}"
                    sh "git checkout ${params.PR_BRANCH} || git checkout -b ${params.PR_BRANCH}"
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv("${env.SONAR_SERVER_NAME}") {
                    sh """
                        ${env.SONAR_SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=django-app \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=**/migrations/**,venv/**,**/.venv/** \
                        -Dsonar.pullrequest.key=${params.PR_NUMBER} \
                        -Dsonar.pullrequest.branch=${params.PR_BRANCH} \
                        -Dsonar.pullrequest.base=${params.PR_BASE}
                    """
                }
            }
        }
    }
}
