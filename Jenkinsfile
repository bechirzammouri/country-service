pipeline{
    agent any 
    tools {
        maven 'M2_HOME'
    }
    stages {
        stage('checkout code'){
            steps {
                git branch: 'main', url: 'https://github.com/bechirzammouri/country-service.git'
            }
        }

        stage ('compile , test code , package in jar file and store it in maven repository'){
            steps {
                sh 'mvn clean install'
            }
            post {
                success {
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis'){
            steps {
                withSonarQubeEnv(installationName: 'MySonarQubeServer', credentialsId: 'sonarqube-token')  {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=country-service -Dsonar.projectName=country-service -Dsonar.host.url=http://172.17.96.1:9000 '
                }
            }
        }
        stage('Deploy to Nexus'){
            steps {
                sh 'mvn deploy -DskipTests -Dmaven.install.skip=true'
            }
        }
        
        stage('Download Artifact from Nexus'){
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-admin-account', 
                                          usernameVariable: 'NEXUS_USER', 
                                          passwordVariable: 'NEXUS_PASS')]) {
                    sh '''#!/bin/bash
                        # Define variables
                        GROUP_ID=net.bechir
                        ARTIFACT_ID=service-country
                        VERSION=0.0.1-SNAPSHOT
                        PACKAGING=war
                        NEXUS_URL=http://172.17.96.1:8081/repository/maven-snapshots
                        
                        # Convert GROUP_ID dots to slashes (e.g., net.bechir -> net/bechir)
                        GROUP_PATH=$(echo ${GROUP_ID} | tr '.' '/')
                        
                        # Download artifact from Nexus
                        echo "Downloading ${ARTIFACT_ID}-${VERSION}.${PACKAGING} from Nexus..."
                        
                        curl -u ${NEXUS_USER}:${NEXUS_PASS} -o ${ARTIFACT_ID}.${PACKAGING} \
                        "${NEXUS_URL}/${GROUP_PATH}/${ARTIFACT_ID}/${VERSION}/${ARTIFACT_ID}-${VERSION}.${PACKAGING}"
                        
                        # Verify download
                        ls -lh ${ARTIFACT_ID}.${PACKAGING}
                    '''
                }
            }
        }
        
        stage('Deploy to Tomcat'){
            steps {
                sh '''
                    # Deploy to Tomcat Manager
                    echo "Deploying service-country.war to Tomcat..."
                    curl -v -u deployer:deployer123 \
                    --upload-file service-country.war \
                    "http://localhost:9090/manager/text/deploy?path=/country-service&update=true"
                '''
            }
        }
    }
}
