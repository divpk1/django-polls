pipeline {
    agent any

    environment {
        // Match the name you gave to your SonarQube installation in Jenkins System Config
        SONAR_SERVER_NAME = 'SonarQube-Server'
        // Name of the scanner tool configured under Global Tool Configuration
        SONAR_SCANNER_HOME = tool 'SonarQubeScanner'
    }

    triggers {
        // Triggers the pipeline when a PR is raised/updated (adjust depending on your plugin setup)
        issueCommentTrigger('.*test-sonar.*') 
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Django Lint & Test') {
            steps {
                echo 'Running unit tests and generating coverage data...'
                // If you are using Docker inside your pipeline to isolate builds:
                sh '''
                    pip install -r requirements.txt
                    pip install coverage
                    coverage run --source='.' manage.py test
                    coverage xml -o coverage.xml
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Wraps the execution with the SonarQube environment details & authentication tokens
                withSonarQubeEnv("${env.SONAR_SERVER_NAME}") {
                    
                    // Conditionally injection PR variables if it's a GitHub/GitLab pull request hook
                    script {
                        def prProperties = ""
                        if (env.CHANGE_ID) { // CHANGE_ID is automatically populated by Multibranch Pipelines for PRs
                            prProperties = """
                                -Dsonar.pullrequest.key=${env.CHANGE_ID} \
                                -Dsonar.pullrequest.branch=${env.CHANGE_BRANCH} \
                                -Dsonar.pullrequest.base=${env.CHANGE_TARGET}
                            """
                        }
                        
                        // Execute the scanner
                        sh """
                            ${env.SONAR_SCANNER_HOME}/bin/sonar-scanner \
                            -Dsonar.projectKey=django-web-app \
                            -Dsonar.projectName="Django Web App" \
                            -Dsonar.sources=. \
                            -Dsonar.exclusions=**/migrations/**,manage.py,venv/**,*.json \
                            -Dsonar.tests=. \
                            -Dsonar.test.inclusions=**/tests/**,test_*.py \
                            -Dsonar.python.coverage.reportPaths=coverage.xml \
                            ${prProperties}
                        """
                    }
                }
            }
        }

        stage("Quality Gate Hook") {
            steps {
                // Wait for SonarQube to finish analysis and return green/red status back to Jenkins
                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline failed due to Quality Gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
    }
}
