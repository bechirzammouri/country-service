pipeline {
    agent any
    
    tools {
        maven 'Maven-3.9' // Configure this name in Jenkins Global Tool Configuration
        jdk 'JDK-21'      // Configure this name in Jenkins Global Tool Configuration
    }
    
    environment {
        // SonarQube
        SONAR_PROJECT_KEY = 'country-service'
        SONAR_HOST_URL = 'http://localhost:9000'
        
        // Nexus
        NEXUS_URL = 'http://localhost:8081'
        NEXUS_REPOSITORY = 'maven-releases'
        NEXUS_CREDENTIAL_ID = 'nexus-credentials'
        
        // Tomcat
        TOMCAT_URL = 'http://localhost:8080'
        TOMCAT_CREDENTIAL_ID = 'tomcat-credentials'
        TOMCAT_DEPLOY_PATH = '/manager/text/deploy'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '🔄 Checking out code from SCM...'
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo '🔨 Building the project...'
                sh 'mvn clean compile'
            }
        }
        
        stage('Unit Tests') {
            steps {
                echo '🧪 Running unit tests...'
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package') {
            steps {
                echo '📦 Packaging the application...'
                sh 'mvn package -DskipTests'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo '🔍 Running SonarQube analysis...'
                withSonarQubeEnv('SonarQube') { // Configure 'SonarQube' server in Jenkins
                    sh """
                        mvn sonar:sonar ^
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} ^
                          -Dsonar.host.url=${SONAR_HOST_URL}
                    """
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                echo '🚦 Waiting for SonarQube Quality Gate...'
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Generate CNES Report') {
            steps {
                echo '📊 Generating CNES SonarQube report...'
                sh """
                    java -jar sonar-cnes-report.jar ^
                      -s ${SONAR_HOST_URL} ^
                      -t %SONAR_TOKEN% ^
                      -p ${SONAR_PROJECT_KEY} ^
                      -o target/sonar-reports
                """
            }
        }
        
        stage('Deploy to Nexus') {
            steps {
                echo '📤 Deploying artifacts to Nexus...'
                script {
                    // Deploy JAR/WAR artifact
                    sh 'mvn deploy -DskipTests'
                    
                    // Deploy CNES Report to Nexus
                    withCredentials([usernamePassword(
                        credentialsId: "${NEXUS_CREDENTIAL_ID}",
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASS'
                    )]) {
                        def reportFile = findFiles(glob: 'target/sonar-reports/*.docx')[0]
                        if (reportFile) {
                            sh """
                                curl -v -u %NEXUS_USER%:%NEXUS_PASS% ^
                                  --upload-file ${reportFile.path} ^
                                  ${NEXUS_URL}/repository/${NEXUS_REPOSITORY}/com/countryservice/sonar-reports/${env.BUILD_NUMBER}/${reportFile.name}
                            """
                        }
                    }
                }
            }
        }
        
        stage('Deploy to Tomcat') {
            steps {
                echo '🚀 Deploying WAR to Tomcat...'
                script {
                    def warFile = findFiles(glob: 'target/*.war')[0]
                    
                    if (warFile) {
                        withCredentials([usernamePassword(
                            credentialsId: "${TOMCAT_CREDENTIAL_ID}",
                            usernameVariable: 'TOMCAT_USER',
                            passwordVariable: 'TOMCAT_PASS'
                        )]) {
                            sh """
                                curl -v -u %TOMCAT_USER%:%TOMCAT_PASS% ^
                                  --upload-file ${warFile.path} ^
                                  "${TOMCAT_URL}/manager/text/deploy?path=/country-service&update=true"
                            """
                        }
                    } else {
                        error 'No WAR file found in target directory!'
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline completed successfully!'
            archiveArtifacts artifacts: 'target/*.jar,target/*.war,target/sonar-reports/**', fingerprint: true
        }
        failure {
            echo '❌ Pipeline failed!'
        }
        always {
            echo '🧹 Cleaning up workspace...'
            cleanWs()
        }
    }
}
