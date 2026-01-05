
 Jenkins & SonarQube Integration Project

![SonarQube Analysis](https://img.shields.io/badge/SonarQube-Passed-success)
![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-red)
![Java](https://img.shields.io/badge/Java-17-orange)

## 📋 Table of Contents
- [What is SonarQube?](#what-is-sonarqube)
- [Why Use SonarQube?](#why-use-sonarqube)
- [Project Overview](#project-overview)
- [Prerequisites](#prerequisites)
- [Installation Guide](#installation-guide)
- [Jenkins Configuration](#jenkins-configuration)
- [Pipeline Execution](#pipeline-execution)
- [Understanding Results](#understanding-results)
- [Troubleshooting](#troubleshooting)

---

## 🔍 What is SonarQube?

**SonarQube** is an open-source platform for **continuous inspection of code quality**. It performs automatic reviews of code to detect:
- **Bugs** 🐛
- **Code Smells** 👃 (maintainability issues)
- **Security Vulnerabilities** 🔒
- **Code Coverage** 📊
- **Code Duplications** 📑

Think of SonarQube as a **code quality doctor** that diagnoses problems in your codebase before they reach production.

---

## 🎯 Why Use SonarQube?

### Real-World Benefits:
1. **Catch Bugs Early** - Find issues before customers do
2. **Improve Code Quality** - Maintain clean, readable code
3. **Security Analysis** - Detect vulnerabilities (SQL injection, XSS, etc.)
4. **Technical Debt Tracking** - Measure time needed to fix issues
5. **Team Standards** - Enforce coding standards across the team
6. **CI/CD Integration** - Automated quality gates in your pipeline

### Industry Usage:
Companies like **Netflix**, **Samsung**, **NASA**, and **Deutsche Bank** use SonarQube to maintain code quality at scale.

---

## 📦 Project Overview

This project demonstrates a complete **DevOps CI/CD pipeline** that:
1. Pulls code from GitHub repository
2. Compiles the Java Maven project
3. Runs unit tests
4. Performs SonarQube code analysis
5. Displays quality metrics and bugs

**Repository:** [https://github.com/divy1436/Boardgame.git](https://github.com/divy1436/Boardgame.git)

### Pipeline Stages:
```
Git Checkout → Compilation → Testing → SonarQube Analysis
   (520ms)       (4s)         (25s)          (48s)
```

---

## ✅ Prerequisites

Before starting, ensure you have:

| Requirement | Version | Purpose |
|------------|---------|---------|
| Linux VM | Ubuntu 20.04/22.04 | Host environment |
| RAM | Minimum 4GB | SonarQube requires memory |
| Java | OpenJDK 17 | Jenkins & SonarQube runtime |
| Root Access | sudo privileges | Installation permissions |

---

## 🚀 Installation Guide

### Step 1: Install Java

SonarQube and Jenkins both require Java.

```bash
# Update system packages
sudo apt update

# Install Java 17
sudo apt install fontconfig openjdk-17-jre -y

# Verify installation
java -version
```

**Expected Output:**
```
openjdk version "17.0.x" 2024-xx-xx
OpenJDK Runtime Environment...
```

---

### Step 2: Install SonarQube

#### 2.1 Create SonarQube User
SonarQube cannot run as root for security reasons.

```bash
# Create dedicated user
sudo adduser sonar
# Enter password when prompted (e.g., sonar123)
```

#### 2.2 Download and Setup

```bash
# Download SonarQube Community Edition
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip

# Install unzip utility if needed
sudo apt install unzip -y

# Extract the archive
unzip sonarqube-9.9.0.65466.zip

# Move to /opt directory
sudo mv sonarqube-9.9.0.65466 /opt/sonarqube

# Change ownership to sonar user
sudo chown -R sonar:sonar /opt/sonarqube
```

#### 2.3 Start SonarQube Service

```bash
# Switch to sonar user
su - sonar

# Navigate to bin directory
cd /opt/sonarqube/bin/linux-x86-64/

# Start SonarQube
./sonar.sh start

# Check status
./sonar.sh status
```

**Expected Output:**
```
SonarQube is running (PID xxxxx).
```

#### 2.4 Access SonarQube Web Interface

1. Open browser: `http://<your-vm-ip>:9000`
2. **Default credentials:**
   - Username: `admin`
   - Password: `admin`
3. You'll be prompted to **change the password** immediately
   - Set new password (e.g., `admin123`)

#### 2.5 Generate Authentication Token

This token allows Jenkins to communicate with SonarQube securely.

1. Click **User Icon** (top right) → **My Account**
2. Go to **Security** tab
3. **Generate Tokens** section:
   - Token Name: `jenkins-token`
   - Type: **User Token**
   - Click **Generate**
4. **⚠️ IMPORTANT:** Copy the token immediately! Example:
   ```
   squ_a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
   ```
   You won't be able to see it again.

---

### Step 3: Install Jenkins

Return to your terminal with sudo access (not the `sonar` user).

```bash
# Add Jenkins GPG key
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

# Add Jenkins repository
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Update and install
sudo apt update
sudo apt install jenkins -y

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Check status
sudo systemctl status jenkins
```

#### 3.1 Unlock Jenkins

```bash
# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

**Copy the password**, then:

1. Open browser: `http://<your-vm-ip>:8080`
2. Paste the password
3. Click **Install suggested plugins**
4. Create your **Admin User** (e.g., username: `admin`, password: `admin123`)
5. Click **Save and Continue** → **Start using Jenkins**

---

## ⚙️ Jenkins Configuration

### Step 4: Install Required Plugins

1. Go to **Dashboard** → **Manage Jenkins** → **Plugins**
2. Click **Available plugins**
3. Search and install:
   - ✅ **SonarQube Scanner**
   - ✅ **Maven Integration** (if not already installed)
4. Click **Install** and restart Jenkins if prompted

---

### Step 5: Add SonarQube Server to Jenkins

This connects Jenkins to your SonarQube instance.

#### 5.1 Configure System Settings

1. **Manage Jenkins** → **System**
2. Scroll to **SonarQube servers** section
3. Click **Add SonarQube**
4. Configure:
   - **Name:** `sonarqube` ⚠️ (Must match pipeline code!)
   - **Server URL:** `http://<your-vm-ip>:9000`
   - **Server authentication token:** Click **Add** → **Jenkins**

#### 5.2 Add SonarQube Credentials

In the popup window:
- **Kind:** Secret text
- **Scope:** Global
- **Secret:** Paste the token you generated earlier
  ```
  squ_a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
  ```
- **ID:** `sonar-token`
- **Description:** `SonarQube Authentication Token`
- Click **Add**

5. Back in SonarQube servers section, select `sonar-token` from the dropdown
6. Click **Apply** and **Save**

---

### Step 6: Configure Tools

Jenkins needs to know where Maven and SonarQube Scanner are located.

1. **Manage Jenkins** → **Tools**

#### 6.1 Maven Configuration

1. Scroll to **Maven installations**
2. Click **Add Maven**
3. Configure:
   - **Name:** `maven` ⚠️ (Must match pipeline code!)
   - ✅ Check **Install automatically**
   - **Version:** Select latest (e.g., 3.9.6)

#### 6.2 SonarQube Scanner Configuration

1. Scroll to **SonarQube Scanner installations**
2. Click **Add SonarQube Scanner**
3. Configure:
   - **Name:** `sonar-scanner` ⚠️ (Must match pipeline code!)
   - ✅ Check **Install automatically**
   - **Version:** Select latest (e.g., 5.0.1)

4. Click **Apply** and **Save**

---

## 🏗️ Pipeline Execution

### Step 7: Create Jenkins Pipeline Job

1. Go to **Jenkins Dashboard**
2. Click **New Item**
3. Enter name: `BoardGame`
4. Select **Pipeline**
5. Click **OK**

### Step 8: Configure Pipeline

1. Scroll to **Pipeline** section
2. **Definition:** Pipeline script
3. Paste the following Groovy script:

```groovy
pipeline {
    agent any
    
    tools {
        maven 'maven' 
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/divy1436/Boardgame.git'
            }
        }
        
        stage('Compilation') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Testing') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """ 
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=BoardGame \
                    -Dsonar.projectKey=BoardGame \
                    -Dsonar.java.binaries=target/classes 
                    """
               }
            }
        }
    }
}
```

4. Click **Save**

### Step 9: Run the Pipeline

1. Click **Build Now** on the left sidebar
2. Watch the **Stage View** in the main panel:
   ```
   ✅ Git Checkout (520ms)
   ✅ Compilation (4s)
   ✅ Testing (25s)
   ✅ SonarQube Analysis (48s)
   ```

3. Click on the build number (e.g., **#2**) under **Build History**
4. Click **Console Output** to see detailed logs

**Expected Final Message:**
```
INFO: ANALYSIS SUCCESSFUL
INFO: ------------------------------------------------------------------------
Finished: SUCCESS
```

---
<img width="1916" height="788" alt="image" src="https://github.com/user-attachments/assets/97fd43f9-9c69-4e89-8155-1aaa5c1e75dd" />


## 📊 Understanding Results

### Step 10: View SonarQube Dashboard

1. Go to SonarQube: `http://<your-vm-ip>:9000`
2. You'll see the **BoardGame** project
3. Click on it to view detailed analysis

### Quality Gate Status: ✅ **PASSED**

Your project passed because all conditions were met!

### Metrics Breakdown:

| Metric | Value | Rating | What It Means |
|--------|-------|--------|---------------|
| **Bugs** | 15 | C (Reliability) | Minor logic errors that should be fixed |
| **Vulnerabilities** | 0 | A (Security) | No security issues found ✅ |
| **Security Hotspots** | 1 (0.0% reviewed) | E (Security Review) | Code needs manual review for potential risks |
| **Code Smells** | 47 | - | Maintainability issues (naming, complexity) |
| **Technical Debt** | 4h 38min | A (Maintainability) | Estimated time to fix all issues |
<img width="1915" height="775" alt="image" src="https://github.com/user-attachments/assets/5c8e3302-755b-405a-91a9-ca06aa49fa62" />


### What Each Term Means:

#### 🐛 **Bugs** (15 found)
Code that is likely to behave unexpectedly or incorrectly. Examples:
- Null pointer dereferences
- Resource leaks
- Incorrect logic

**In your case:** You have 15 bugs that need fixing to improve reliability.

#### 🔒 **Vulnerabilities** (0 found)
Security issues that could be exploited. Examples:
- SQL injection
- Cross-site scripting (XSS)
- Weak cryptography

**In your case:** Excellent! No security vulnerabilities detected.

#### ⚠️ **Security Hotspots** (1 found, 0% reviewed)
Code that needs manual review for security. Examples:
- File uploads without validation
- Dynamic SQL queries
- Cookie handling

**Action needed:** Review the flagged code manually.

#### 👃 **Code Smells** (47 found)
Maintainability issues that make code harder to understand/modify. Examples:
- Long methods
- Duplicated code
- Complex conditions

**In your case:** 47 areas where code could be cleaner.

#### ⏱️ **Technical Debt** (4h 38min)
Estimated time to fix all code smells.

---

## 🔧 Troubleshooting

### Common Issues and Solutions

#### Issue 1: SonarQube Won't Start
**Error:** `Wrapper cannot be started...`

**Solution:**
```bash
# Increase virtual memory
sudo sysctl -w vm.max_map_count=262144

# Make permanent
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

#### Issue 2: Jenkins Can't Connect to SonarQube
**Error:** `Fail to get bootstrap index from server`

**Solution:**
- Verify SonarQube is running: `http://<ip>:9000`
- Check server URL in Jenkins (should not have trailing slash)
- Verify authentication token is correct

#### Issue 3: Maven Compilation Fails
**Error:** `JAVA_HOME is not defined`

**Solution:**
```bash
# Find Java path
which java

# Set in Jenkins
# Manage Jenkins → System → Global properties
# Add environment variable:
# Name: JAVA_HOME
# Value: /usr/lib/jvm/java-17-openjdk-amd64
```

#### Issue 4: SonarQube Analysis Fails
**Error:** `java.binaries property is not set`

**Solution:**
Ensure your SonarQube stage includes:
```groovy
-Dsonar.java.binaries=target/classes
```

---

## 🎓 Learning Resources

### Official Documentation:
- [SonarQube Documentation](https://docs.sonarqube.org/)
- [Jenkins Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [Maven Guide](https://maven.apache.org/guides/)

### Recommended Next Steps:
1. ✅ Fix the 15 bugs reported by SonarQube
2. ✅ Review the security hotspot
3. ✅ Add quality gate status badge to your README
4. ✅ Configure Quality Gates to fail builds on critical issues
5. ✅ Integrate with GitHub webhooks for automatic triggers

---

## 📝 Project Structure

```
Boardgame/
├── src/
│   ├── main/
│   │   └── java/          # Application source code
│   └── test/
│       └── java/          # Unit tests
├── target/
│   └── classes/           # Compiled bytecode (used by SonarQube)
├── pom.xml                # Maven configuration
└── Jenkinsfile            # Pipeline script
```

---

## 🏆 Success Criteria

Your pipeline is successfully configured if:
- ✅ All 4 stages complete without errors
- ✅ SonarQube shows "Passed" quality gate
- ✅ Project appears in SonarQube dashboard
- ✅ You can view detailed code metrics
- ✅ Build time is approximately 1 min 38 sec

---

## 📧 Support

If you encounter issues:
1. Check the **Console Output** in Jenkins
2. Review **SonarQube logs**: `/opt/sonarqube/logs/sonar.log`
3. Verify all tool names match exactly (case-sensitive!)

---

## 📜 License

This project is for educational purposes. Original repository by [divy1436](https://github.com/divy1436).

---

## ⭐ Conclusion

Congratulations! You've successfully built a **production-grade CI/CD pipeline** that:
- Automates code quality checks
- Detects bugs and vulnerabilities before production
- Provides actionable metrics for improvement
- Integrates industry-standard DevOps tools

**Next Challenge:** Can you reduce the 15 bugs to 0? 🎯

---

**Made with ❤️ for DevOps Engineers**
