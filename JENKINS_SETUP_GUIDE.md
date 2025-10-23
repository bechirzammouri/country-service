# Jenkins Pipeline Setup Guide for Country Service

## 📋 Overview

This guide will help you configure Jenkins to:

1. Build and test the Country Service application
2. Run SonarQube analysis with quality gates
3. Deploy artifacts (.jar/.war) to Nexus Repository
4. Deploy the application to Tomcat server

---

## 🔧 Prerequisites

### Required Software

- ✅ Jenkins (with Pipeline plugin)
- ✅ Maven 3.9+
- ✅ JDK 21
- ✅ SonarQube Server (running on http://localhost:9000)
- ✅ Nexus Repository Manager (running on http://localhost:8081)
- ✅ Apache Tomcat (running on http://localhost:8080)
- ✅ CNES Report Plugin JAR (optional, for detailed reports)

---

## 📦 Step 1: Install Required Jenkins Plugins

Navigate to: **Jenkins → Manage Jenkins → Manage Plugins → Available**

Install these plugins:

1. **Pipeline** (for Jenkinsfile support)
2. **Pipeline: Stage View** (for visual pipeline stages)
3. **Git** (for source control)
4. **Maven Integration**
5. **SonarQube Scanner**
6. **Nexus Artifact Uploader**
7. **Deploy to container** (for Tomcat deployment)
8. **JUnit** (for test reports)
9. **JaCoCo** (for code coverage)

After installation, restart Jenkins.

---

## 🛠️ Step 2: Configure Global Tools

### A. Configure Maven

1. Go to: **Jenkins → Manage Jenkins → Global Tool Configuration**
2. Scroll to **Maven** section
3. Click **Add Maven**
4. Configure:
   - Name: `Maven-3.9`
   - Install automatically: ✅ (Check)
   - Version: `3.9.11` (or your installed version)
5. Click **Save**

### B. Configure JDK

1. In the same **Global Tool Configuration** page
2. Scroll to **JDK** section
3. Click **Add JDK**
4. Configure:
   - Name: `JDK-21`
   - JAVA_HOME: `C:\Program Files\Java\jdk-21` (adjust to your path)
   - Or check "Install automatically"
5. Click **Save**

---

## 🔐 Step 3: Configure Jenkins Credentials

### A. SonarQube Token

1. Go to: **Jenkins → Manage Jenkins → Manage Credentials**
2. Click on **(global)** domain
3. Click **Add Credentials**
4. Configure:
   - Kind: `Secret text`
   - Secret: `[Your SonarQube Token]`
   - ID: `sonarqube-token`
   - Description: `SonarQube Authentication Token`
5. Click **OK**

### B. Nexus Credentials

1. Click **Add Credentials** again
2. Configure:
   - Kind: `Username with password`
   - Username: `admin` (default Nexus username)
   - Password: `admin123` (default Nexus password)
   - ID: `nexus-credentials`
   - Description: `Nexus Repository Credentials`
3. Click **OK**

### C. Tomcat Credentials

1. Click **Add Credentials** again
2. Configure:
   - Kind: `Username with password`
   - Username: `admin` (Tomcat manager username)
   - Password: `admin` (Tomcat manager password)
   - ID: `tomcat-credentials`
   - Description: `Tomcat Manager Credentials`
3. Click **OK**

---

## 🌐 Step 4: Configure SonarQube Server in Jenkins

1. Go to: **Jenkins → Manage Jenkins → Configure System**
2. Scroll to **SonarQube servers** section
3. Click **Add SonarQube**
4. Configure:
   - Name: `SonarQube`
   - Server URL: `http://localhost:9000`
   - Server authentication token: Select `sonarqube-token` (from Step 3A)
5. Click **Save**

---

## 📝 Step 5: Create Jenkins Pipeline Job

### A. Create New Pipeline

1. Go to Jenkins Dashboard
2. Click **New Item**
3. Enter item name: `country-service-pipeline`
4. Select **Pipeline**
5. Click **OK**

### B. Configure Pipeline

1. In the **General** section:

   - Description: `CI/CD Pipeline for Country Service`

2. In the **Build Triggers** section (optional):

   - ✅ Poll SCM: `H/5 * * * *` (check every 5 minutes)
   - Or ✅ GitHub hook trigger for GITScm polling

3. In the **Pipeline** section:

   - Definition: `Pipeline script from SCM`
   - SCM: `Git`
   - Repository URL: `[Your Git Repository URL]`
   - Branch: `*/main` (or your branch)
   - Script Path: `Jenkinsfile`

4. Click **Save**

---

## 🚀 Step 6: Configure Nexus Repository

### A. Create Repositories in Nexus

1. Open Nexus: http://localhost:8081
2. Login with admin credentials
3. Go to: **Settings (⚙️) → Repositories**
4. Verify these repositories exist:
   - `maven-releases` (hosted, release)
   - `maven-snapshots` (hosted, snapshot)
   - `maven-public` (group, proxy to central)

### B. Configure Nexus Realms

1. Go to: **Settings → Security → Realms**
2. Add **npm Bearer Token Realm** to Active
3. Add **Docker Bearer Token Realm** to Active (if using Docker)
4. Click **Save**

---

## 🌐 Step 7: Configure Tomcat Manager

### A. Edit tomcat-users.xml

Location: `[TOMCAT_HOME]/conf/tomcat-users.xml`

Add these roles and users:

```xml
<tomcat-users>
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <role rolename="manager-jmx"/>
  <role rolename="manager-status"/>

  <user username="admin" password="admin" roles="manager-gui,manager-script"/>
</tomcat-users>
```

### B. Edit context.xml (Allow Remote Access)

Location: `[TOMCAT_HOME]/webapps/manager/META-INF/context.xml`

Comment out the Valve restriction:

```xml
<!-- <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
```

### C. Restart Tomcat

```powershell
# Stop Tomcat
[TOMCAT_HOME]\bin\shutdown.bat

# Start Tomcat
[TOMCAT_HOME]\bin\startup.bat
```

---

## 📥 Step 8: Download CNES Report Plugin (Optional)

1. Download from: https://github.com/cnescatlab/sonar-cnes-report/releases
2. Download: `sonar-cnes-report-[version].jar`
3. Place in project root or Jenkins workspace
4. Update Jenkinsfile path if needed

---

## 🧪 Step 9: Test the Pipeline

### A. Run Pipeline Manually

1. Go to your pipeline job: `country-service-pipeline`
2. Click **Build Now**
3. Watch the **Stage View** for progress

### B. Verify Each Stage

- ✅ **Checkout**: Code pulled from Git
- ✅ **Build**: Maven compiles successfully
- ✅ **Unit Tests**: All tests pass (check test results)
- ✅ **Package**: JAR/WAR created in `target/`
- ✅ **SonarQube Analysis**: Analysis sent to SonarQube
- ✅ **Quality Gate**: Quality gate passes
- ✅ **Generate CNES Report**: Report generated (if configured)
- ✅ **Deploy to Nexus**: Artifacts uploaded to Nexus
- ✅ **Deploy to Tomcat**: Application deployed to Tomcat

### C. Verify Deployments

**Check Nexus:**

```powershell
# Open browser
Start-Process http://localhost:8081
```

Navigate to: **Browse → maven-releases** → Find your artifact

**Check SonarQube:**

```powershell
# Open browser
Start-Process http://localhost:9000/dashboard?id=country-service
```

**Check Tomcat:**

```powershell
# Open browser
Start-Process http://localhost:8080/country-service
```

---

## 🔍 Troubleshooting

### Issue 1: SonarQube Quality Gate Fails

**Solution:**

- Check SonarQube dashboard for issues
- Adjust quality gate thresholds in SonarQube
- Or add `abortPipeline: false` in Jenkinsfile

### Issue 2: Nexus Deployment Fails (401 Unauthorized)

**Solution:**

- Verify credentials in Jenkins
- Check Maven settings.xml has correct Nexus credentials
- Verify repository IDs match between pom.xml and settings.xml

### Issue 3: Tomcat Deployment Fails (403 Forbidden)

**Solution:**

- Verify tomcat-users.xml has manager-script role
- Check context.xml allows remote access
- Restart Tomcat after changes

### Issue 4: Maven Build Fails

**Solution:**

- Check Java version (must be JDK 21)
- Run `mvn clean install` locally first
- Check for dependency conflicts

### Issue 5: Tests Fail

**Solution:**

- Verify H2 database dependency is present
- Check test configuration in application.properties
- Run tests locally: `mvn test`

---

## 📊 Monitoring & Reports

### Jenkins Console Output

- View detailed logs: **Pipeline → Build History → Console Output**

### Test Reports

- View in Jenkins: **Pipeline → Test Result**
- HTML Report: `target/surefire-reports/`

### Code Coverage (JaCoCo)

- View in Jenkins: **Pipeline → Code Coverage**
- HTML Report: `target/site/jacoco/index.html`

### SonarQube Dashboard

- URL: http://localhost:9000/dashboard?id=country-service
- Metrics: Bugs, Vulnerabilities, Code Smells, Coverage, Duplications

### Nexus Browse

- URL: http://localhost:8081/#browse/browse
- View uploaded artifacts with version history

---

## 🔄 Pipeline Workflow Summary

```
1. Checkout Code (Git)
       ↓
2. Build (mvn compile)
       ↓
3. Unit Tests (mvn test)
       ↓
4. Package (mvn package)
       ↓
5. SonarQube Analysis (mvn sonar:sonar)
       ↓
6. Quality Gate Check
       ↓
7. Generate CNES Report (optional)
       ↓
8. Deploy to Nexus (mvn deploy + curl for reports)
       ↓
9. Deploy to Tomcat (curl POST to manager)
       ↓
10. Archive Artifacts (Jenkins)
```

---

## 📚 Additional Resources

- Jenkins Pipeline Syntax: https://www.jenkins.io/doc/book/pipeline/syntax/
- Maven Deploy Plugin: https://maven.apache.org/plugins/maven-deploy-plugin/
- SonarQube Documentation: https://docs.sonarqube.org/
- Nexus Repository Manager: https://help.sonatype.com/repomanager3
- Tomcat Manager: https://tomcat.apache.org/tomcat-9.0-doc/manager-howto.html

---

## ✅ Checklist

Before running the pipeline, ensure:

- [ ] Jenkins plugins installed
- [ ] Maven and JDK configured in Jenkins
- [ ] All credentials added to Jenkins
- [ ] SonarQube server configured in Jenkins
- [ ] SonarQube is running (http://localhost:9000)
- [ ] Nexus is running (http://localhost:8081)
- [ ] Tomcat is running (http://localhost:8080)
- [ ] Tomcat manager configured with correct users
- [ ] pom.xml has distributionManagement section
- [ ] Maven settings.xml has Nexus credentials
- [ ] Jenkinsfile is in project root
- [ ] Git repository is accessible

---

## 🎉 Success Indicators

After successful pipeline execution:

1. ✅ All Jenkins stages show green
2. ✅ Test reports show 0 failures
3. ✅ SonarQube shows "Passed" quality gate
4. ✅ Artifact visible in Nexus repository
5. ✅ Application accessible at http://localhost:8080/country-service
6. ✅ Jenkins artifacts archived

---

**Good luck with your CI/CD pipeline! 🚀**
