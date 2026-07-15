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
        // The automatic "Declarative: Checkout SCM" stage handles getting the repo.
        // We added a step below to ensure we are explicitly checked out to the PR branch.
        stage('Prepare Branch') {
            steps {
                script {
                    echo "Ensuring workspace is on branch: ${params.PR_BRANCH}"
                    // Switches the already cloned repo to the exact branch passed by GitHub Actions
                    sh "git checkout ${params.PR_BRANCH} || git checkout -b ${params.PR_BRANCH}"
                }
            }
        }

        stage('Django Test Matrix') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
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
