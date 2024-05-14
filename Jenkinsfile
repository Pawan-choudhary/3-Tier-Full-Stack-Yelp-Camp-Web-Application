pipeline {
    agent any 
    tools {
        nodejs 'node21'
    }
    environment {
        SCANNER_HOME=tool "sonar-scanner"
    }

    stages {
        stage("Git Checkout") {
            steps {
                git(url: 'https://github.com/Pawan-choudhary/3-Tier-Full-Stack-Yelp-Camp-Web-Application.git', branch: 'main')
            }
        }
        stage('Dependency Check') {
            steps {
                script {
                    // Run npm audit to check for vulnerabilities
                    def npmAudit = sh(script: 'npm audit --json', returnStdout: true).trim()
                    def auditJson = readJSON text: npmAudit

                    // Check if any vulnerabilities were found
                    if (auditJson.metadata.vulnerabilities.totalVulnerabilities > 0) {
                        currentBuild.result = 'FAILURE'
                        error 'npm audit found vulnerabilities. Build failed.'
                    }
                }
            }
        }
        stage("Install Dependencies") {
            steps {
                sh "npm install"
            }
        }
        stage('Run Tests') {
            steps {
                // Run npm test to execute tests
                sh "npm test"
            }
        }
        
        stage("Trivy FS Scan") {
            steps {
                sh "trivy fs --formate table -o fs-report.html ."
            }
        }
        
        
        stage("SonarQube") {
            steps {
                withSonarQubeEnv('sonar') {
                sh "SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Campground -Dsonar.projectName="
                }
            }
        }
        stage("Docker Build and tag") {
            steps {
                script {
                    withDockerRegistry(credentialsID:'docker-cred', toolName: 'docker') {
                        sh "docker buid -t pawanchoudharynain/camp:latest ."
                    }
                }
            }
        }
        stage("Trivy Image Scan") {
            steps {
                sh "trivy image --formate table -o fs-report.html pawanchoudharynain/camp:latest"
            }
        }
        stage("Push Docker Image") {
            steps {
                script {
                    withDockerRegistry(credentialsID:'docker-cred', toolName: 'docker') {
                        sh "docker push pawanchoudharynain/camp:latest"
                    }
                }
            }  
        }

        stage("Docker deploy to Dev") {
            steps {
                script {
                    withDockerRegistry(credentialsID:'docker-cred', toolName: 'docker') {
                        sh "docker run -d -p 3000:3000 pawanchoudharynain/camp:latest"
                    }
                }   
            }  
        }

        
    }
    post {
        success {
            // Send success notification to developer
            emailext body: 'The Jenkins pipeline succeeded.',
                     subject: 'Pipeline Success',
                     to: 'developer@example.com'
        }
        failure {
            // Identify the failed stage using environment variable
            def failedStage = env.STAGE_NAME

            // Log a message indicating the failed stage
            echo "Pipeline failed at stage: ${failedStage}"

            // Optional: Additional actions based on the failed stage
            if (failedStage == 'Unit Tests') {
                // Trigger notification for test failures
                emailext body: "Unit tests failed in pipeline!", subject: "Unit Test Failure - ${JOB_NAME}"
            } else if (failedStage == 'Security Scanning') {
                // Trigger notification for security vulnerabilities
                // ...
            }
        }
    }
}