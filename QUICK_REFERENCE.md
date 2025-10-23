# Quick Reference - Jenkins Pipeline Commands

## 🔧 Maven Commands Used in Pipeline

### Build & Compile

```powershell
mvn clean compile
```

### Run Tests

```powershell
mvn test
```

### Package Application

```powershell
mvn package -DskipTests
```

### SonarQube Analysis

```powershell
mvn sonar:sonar -Dsonar.projectKey=country-service -Dsonar.host.url=http://localhost:9000 -Dsonar.token=YOUR_TOKEN
```

### Deploy to Nexus

```powershell
mvn deploy -DskipTests
```

### Full Pipeline (Manual)

```powershell
mvn clean verify sonar:sonar deploy -Dsonar.projectKey=country-service -Dsonar.token=YOUR_TOKEN
```

---

## 🌐 Nexus Commands

### Upload Artifact with cURL

```powershell
curl -v -u admin:admin123 `
  --upload-file target/service-country-1.0.0.war `
  http://localhost:8081/repository/maven-releases/com/countryservice/service-country/1.0.0/service-country-1.0.0.war
```

### Upload CNES Report

```powershell
curl -v -u admin:admin123 `
  --upload-file target/sonar-reports/report.docx `
  http://localhost:8081/repository/maven-releases/com/countryservice/sonar-reports/1/report.docx
```

### Check if Artifact Exists

```powershell
curl -u admin:admin123 `
  http://localhost:8081/repository/maven-releases/com/countryservice/service-country/1.0.0/
```

---

## 🚀 Tomcat Deployment Commands

### Deploy WAR with cURL

```powershell
curl -v -u admin:admin `
  --upload-file target/service-country-1.0.0.war `
  "http://localhost:8080/manager/text/deploy?path=/country-service&update=true"
```

### Undeploy Application

```powershell
curl -u admin:admin `
  "http://localhost:8080/manager/text/undeploy?path=/country-service"
```

### List Deployed Applications

```powershell
curl -u admin:admin `
  http://localhost:8080/manager/text/list
```

### Start Application

```powershell
curl -u admin:admin `
  "http://localhost:8080/manager/text/start?path=/country-service"
```

### Stop Application

```powershell
curl -u admin:admin `
  "http://localhost:8080/manager/text/stop?path=/country-service"
```

### Reload Application

```powershell
curl -u admin:admin `
  "http://localhost:8080/manager/text/reload?path=/country-service"
```

---

## 📊 SonarQube Commands

### Check Project Status

```powershell
curl -u YOUR_TOKEN: `
  http://localhost:9000/api/qualitygates/project_status?projectKey=country-service
```

### Get Project Metrics

```powershell
curl -u YOUR_TOKEN: `
  "http://localhost:9000/api/measures/component?component=country-service&metricKeys=bugs,vulnerabilities,code_smells,coverage"
```

---

## 🔍 Jenkins CLI Commands

### Trigger Build

```powershell
java -jar jenkins-cli.jar -s http://localhost:8080/ build country-service-pipeline
```

### Get Build Status

```powershell
java -jar jenkins-cli.jar -s http://localhost:8080/ get-job country-service-pipeline
```

### Console Output

```powershell
java -jar jenkins-cli.jar -s http://localhost:8080/ console country-service-pipeline
```

---

## 🧪 Testing Commands

### Run Specific Test Class

```powershell
mvn test -Dtest=ControllerMockMvcTests
```

### Run Tests with Coverage

```powershell
mvn clean test jacoco:report
```

### Skip Tests

```powershell
mvn clean install -DskipTests
```

---

## 🔐 Credentials Management

### Encode Password for Maven settings.xml

```powershell
mvn --encrypt-master-password YOUR_PASSWORD
```

### Encrypt Server Password

```powershell
mvn --encrypt-password YOUR_PASSWORD
```

---

## 📦 Artifact Information

### Show Dependency Tree

```powershell
mvn dependency:tree
```

### Show Effective POM

```powershell
mvn help:effective-pom
```

### Show Project Info

```powershell
mvn help:describe -Dplugin=deploy
```

---

## 🛠️ Troubleshooting Commands

### Clean Maven Cache

```powershell
Remove-Item -Recurse -Force "$env:USERPROFILE\.m2\repository"
```

### Force Update Dependencies

```powershell
mvn clean install -U
```

### Debug Maven Build

```powershell
mvn clean install -X
```

### Check Java Version

```powershell
java -version
mvn -version
```

### Check Tomcat Status

```powershell
netstat -ano | findstr :8080
```

### Check SonarQube Status

```powershell
curl http://localhost:9000/api/system/status
```

### Check Nexus Status

```powershell
curl http://localhost:8081/service/rest/v1/status
```

---

## 📁 Important File Locations

### Jenkins

- **Workspace:** `C:\ProgramData\Jenkins\.jenkins\workspace\country-service-pipeline\`
- **Config:** `C:\ProgramData\Jenkins\.jenkins\config.xml`
- **Credentials:** `C:\ProgramData\Jenkins\.jenkins\credentials.xml`

### Maven

- **Local Repo:** `C:\Users\hp\.m2\repository\`
- **Settings:** `C:\Users\hp\.m2\settings.xml`
- **Global Settings:** `C:\ProgramData\chocolatey\lib\maven\apache-maven-3.9.11\conf\settings.xml`

### Project

- **POM:** `C:\Users\hp\IdeaProjects\service-country\pom.xml`
- **Target:** `C:\Users\hp\IdeaProjects\service-country\target\`
- **Jenkinsfile:** `C:\Users\hp\IdeaProjects\service-country\Jenkinsfile`

---

## 🌐 Important URLs

| Service        | URL                                   | Purpose               |
| -------------- | ------------------------------------- | --------------------- |
| Jenkins        | http://localhost:8080                 | CI/CD Server          |
| SonarQube      | http://localhost:9000                 | Code Quality Analysis |
| Nexus          | http://localhost:8081                 | Artifact Repository   |
| Tomcat         | http://localhost:8080                 | Application Server    |
| Application    | http://localhost:8080/country-service | Deployed Application  |
| Tomcat Manager | http://localhost:8080/manager/html    | Tomcat Web Manager    |

---

## 🎯 Common Pipeline Scenarios

### Scenario 1: Deploy Without Tests

```groovy
stage('Package') {
    steps {
        bat 'mvn package -DskipTests'
    }
}
```

### Scenario 2: Deploy Snapshot Version

```xml
<!-- In pom.xml -->
<version>1.0.0-SNAPSHOT</version>
```

Pipeline will automatically deploy to `nexus-snapshots` repository.

### Scenario 3: Conditional Deployment

```groovy
stage('Deploy to Nexus') {
    when {
        branch 'main'
    }
    steps {
        bat 'mvn deploy -DskipTests'
    }
}
```

### Scenario 4: Parallel Stages

```groovy
stage('Parallel Analysis') {
    parallel {
        stage('SonarQube') {
            steps {
                bat 'mvn sonar:sonar'
            }
        }
        stage('Security Scan') {
            steps {
                bat 'mvn dependency-check:check'
            }
        }
    }
}
```

---

## 📧 Notifications (Add to Jenkinsfile)

### Email Notification

```groovy
post {
    success {
        emailext (
            subject: "✅ Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "Good news! The build was successful.",
            to: "team@example.com"
        )
    }
    failure {
        emailext (
            subject: "❌ Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "Build failed. Check console output.",
            to: "team@example.com"
        )
    }
}
```

### Slack Notification

```groovy
post {
    always {
        slackSend (
            color: currentBuild.result == 'SUCCESS' ? 'good' : 'danger',
            message: "${env.JOB_NAME} #${env.BUILD_NUMBER}: ${currentBuild.result}"
        )
    }
}
```

---

## 🔄 Environment-Specific Deployments

### Development Environment

```groovy
environment {
    TOMCAT_URL = 'http://dev-server:8080'
    NEXUS_REPOSITORY = 'maven-snapshots'
}
```

### Production Environment

```groovy
environment {
    TOMCAT_URL = 'http://prod-server:8080'
    NEXUS_REPOSITORY = 'maven-releases'
}
```

---

**Quick Tip:** Save this file for easy reference during pipeline development! 📚
