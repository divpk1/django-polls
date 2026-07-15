pipeline {
    agent any

    // Defines the incoming parameters sent by the GitHub Action curl command
    parameters {
        string(name: 'PR_NUMBER', defaultValue: '', description: 'GitHub PR Number')
        string(name: 'PR_BRANCH', defaultValue: '', description: 'Source Branch')
        string(name: 'PR_BASE',   defaultValue: '', description: 'Target Base Branch')
    }

    environment {
        SONAR_SERVER_NAME  = 'SonarQube-Public-Server'
        SONAR_SCANNER_HOME = tool 'SonarQubeScanner'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Clones the specific PR branch sent by GitHub Actions
                git url: 'https://github.com/your-username/your-repo.git', branch: "${params.PR_BRANCH}"
            }
        }

        stage('Django Test Matrix') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt || pip install django coverage
                    coverage run --source='.' manage.py test --noinput
                    coverage xml -i -o coverage.xml
                '''
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv("${env.SONAR_SERVER_NAME}") {
                    sh """
                        ${env.SONAR_SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=django-app \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=**/migrations/**,venv/** \
                        -Dsonar.python.coverage.reportPaths=coverage.xml \
                        -Dsonar.pullrequest.key=${params.PR_NUMBER} \
                        -Dsonar.pullrequest.branch=${params.PR_BRANCH} \
                        -Dsonar.pullrequest.base=${params.PR_BASE}
                    """
                }
            }
        }
    }
}
