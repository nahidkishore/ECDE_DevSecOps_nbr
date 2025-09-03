terraform init
terraform plan 
terraform apply 


squ_d1a170220cc89f80f0da4ba72fab0b3ba35794b0













```groovy

pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node20'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
                sh 'rm -rf $HOME/.sonar/cache || true'
            }
        }

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/nahidkishore/Shop-safely-now.git'
            }
        }

        stage('Secrets Scan with Gitleaks') {
            steps {
                echo "🔍 Scanning for secrets using Gitleaks..."
                sh '''
                    gitleaks detect --source . --redact --no-git -v || echo "⚠️ Gitleaks found issues!"
                '''
            }
        }
        
        stage("SonarQube Analysis") {
    steps {
        script {
            echo '🔍 Starting SonarQube Static Code Analysis...'
        }

        withSonarQubeEnv('sonar-server') {
            sh """
                ${SCANNER_HOME}/bin/sonar-scanner \
                -Dsonar.projectKey=Shop-Safely-Now \
                -Dsonar.projectName=Shop-Safely-Now \
                -Dsonar.java.binaries=. \
                -Dsonar.qualitygate.wait=true
            """
        }

        script {
            echo '✅ SonarQube Scanner executed. Waiting for Quality Gate result...'
        }
    }
}

        stage("SonarQube Quality Gate") {
    steps {
        timeout(time: 2, unit: 'MINUTES') {
            script {
                echo '🕒 Waiting for SonarQube Quality Gate result (max 2 minutes)...'

                def qualityGate = waitForQualityGate(
                    abortPipeline: true, 
                    credentialsId: 'sonar-token'
                )

                echo "🛡️ SonarQube Quality Gate Status: ${qualityGate.status}"

                if (qualityGate.status != 'OK') {
                    error "❌ Quality Gate failed: ${qualityGate.status}. Blocking the pipeline."
                } else {
                    echo '✅ Quality Gate passed!'
                }
            }
        }
    }
}


     stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . "
                
            }
        }

        stage('TRIVY FS SCAN') {
            steps {
                   echo "🔍 Running Trivy FS Scan for High & Critical vulnerabilities..."
        sh '''
            trivy fs . \
              --severity HIGH,CRITICAL \
              --format json \
              --output trivy-fs-report.json
        '''
        // Archive JSON report for audit
        archiveArtifacts artifacts: 'trivy-fs-report.json', fingerprint: true
                
            }
        }

        stage("OWASP Dependency Check") {
            steps {
                script {
                    echo 'Starting OWASP Dependency Check...'
                    try {
                        dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP-Check'
                        echo 'OWASP Dependency Check completed successfully.'
                    } catch (Exception e) {
                        error "OWASP Dependency Check failed: ${e.message}"
                    }

                    echo 'Publishing OWASP Dependency Check report...'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                    echo 'OWASP Dependency Check report published successfully.'
                }
            }
        }


stage("Build and Push to Docker Hub") {
            steps {
                echo ' Building and pushing Docker image...'
                withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                    script{
                        def imageTag = "${dockerHubUser}/shop-safely-now:${env.BUILD_NUMBER}"
                        sh """
                        docker build -t ${imageTag} .
                        echo "${dockerHubPassword}" | docker login -u "${dockerHubUser}" --password-stdin
                        docker push ${imageTag}
                        


                        """
                        env.BUILT_IMAGE = imageTag
                    }
                }
            }
        }
        
         stage("TRIVY Docker Image Scan") {
    steps {
        echo 'Scanning built Docker image with Trivy...'
        script {
            def imageToScan = env.BUILT_IMAGE
            if (!imageToScan?.trim()) {
                error "❌ BUILT_IMAGE is not set! Please verify the Build stage."
            }

            sh """
                trivy image --format json --severity CRITICAL,HIGH --output trivy-image-scan.json ${imageToScan} || echo "⚠️ Vulnerabilities found!"

            """
        }
        archiveArtifacts artifacts: 'trivy-image-scan.json', fingerprint: true
    }
}



        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}


```