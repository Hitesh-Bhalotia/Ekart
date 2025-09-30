# Ekart Shopping Cart Application - Jenkins Pipeline

This Jenkins pipeline automates the complete CI/CD process for the Ekart shopping cart application with comprehensive security scanning, code quality analysis, artifact management, and containerized deployment.

## 📄 Project Attribution

**Important Note**: This project is a fork of the original Ekart application created by [jaiswaladi246](https://github.com/jaiswaladi246/Ekart).

- **Original Repository**: https://github.com/jaiswaladi246/Ekart
- **Fork Repository**: https://github.com/Hitesh-Bhalotia/Ekart.git
- **My Contribution**: The Jenkins pipeline configuration and CI/CD automation setup documented in this README

All credit for the Ekart shopping cart application code (frontend, backend, and core functionality) goes to the original author. This documentation focuses specifically on the Jenkins pipeline implementation and DevOps automation added to the forked repository.

## 🏗️ Pipeline Overview

This comprehensive declarative Jenkins pipeline performs the following operations:
- Source code checkout from GitHub
- Maven compilation and packaging
- Security vulnerability scanning (Trivy & OWASP)
- SonarQube code quality analysis
- Artifact deployment to Nexus Repository
- Docker image building and tagging
- Containerized application deployment

## 📋 Prerequisites

### Jenkins Requirements
- Jenkins server with pipeline plugin support
- Required Jenkins plugins (detailed in Plugin Installation section)
- Jenkins credentials configured for Docker Hub and SonarQube

### Tool Configurations
Configure the following tools in Jenkins Global Tool Configuration:

#### JDK Configuration
- **Name**: `jdk`
- **Type**: JDK installation
- **Installation**: Eclipse Temurin installer plugin (automatic)
- **Version**: Java 8 or higher (Java 11+ recommended)

#### Maven Configuration
- **Name**: `maven3`
- **Type**: Maven installation
- **Installation**: Automatic installation
- **Version**: Maven 3.6+ recommended

#### SonarQube Scanner Configuration
- **Name**: `sonar-scanner`
- **Type**: SonarQube Scanner installation
- **Installation**: Automatic installation
- **Version**: Latest stable version

#### Docker Configuration
- **Name**: `docker`
- **Type**: Docker installation
- **Installation**: Automatic installation
- **Integration**: Docker Pipeline plugin integration

#### OWASP Dependency Check
- **Name**: `DC`
- **Type**: Dependency-Check installation
- **Installation**: Automatic installer

### System Dependencies
Ensure the following tools are installed on Jenkins agents:
- **Trivy**: Container and filesystem vulnerability scanner
- **Docker**: For containerized deployment and image building
- **Maven**: Java build automation tool
- **Git**: For source code management

## 🚀 Pipeline Stages

### 1. Git Checkout
```groovy
git branch: 'main', url: 'https://github.com/Hitesh-Bhalotia/Ekart.git'
```
- **Purpose**: Clones the source code from the main branch
- **Repository**: `https://github.com/Hitesh-Bhalotia/Ekart.git`
- **Branch**: `main`
- **Method**: Explicit branch specification for consistency

### 2. Compile
```groovy
sh "mvn compile"
```
- **Purpose**: Compiles Java source code using Maven
- **Tool**: Maven 3 (configured as `maven3`)
- **Output**: Compiled class files in `target/classes`
- **Dependencies**: Downloads and resolves project dependencies

### 3. Trivy Filesystem Scan
```groovy
sh "trivy fs ."
```
- **Purpose**: Performs comprehensive filesystem vulnerability scanning
- **Tool**: Trivy security scanner
- **Scope**: Entire project directory (`.`)
- **Detects**: 
  - CVE vulnerabilities in dependencies
  - Security misconfigurations
  - Exposed secrets and credentials
  - License compliance issues

### 4. OWASP Dependency Check
```groovy
dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DC'
dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
```
- **Purpose**: Scans project dependencies for known vulnerabilities
- **Tool**: OWASP Dependency Check
- **Scope**: Entire project directory (`./`)
- **Output**: Generates `dependency-check-report.xml`
- **Integration**: Results published to Jenkins dashboard with trend analysis

### 5. SonarQube Analysis
```groovy
withSonarQubeEnv('sonar') {
    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=EKART \
    -Dsonar.java.binaries=. \
    -Dsonar.projectKey=EKART '''
}
```
- **Purpose**: Performs static code analysis for quality and security
- **Tool**: SonarQube Scanner
- **Project**: EKART
- **Analysis**: Code coverage, bugs, vulnerabilities, code smells
- **Integration**: Results sent to SonarQube server configured as `sonar`

### 6. Build
```groovy
sh "mvn package -DskipTests=true"
```
- **Purpose**: Packages the application into JAR/WAR file
- **Tool**: Maven package goal
- **Configuration**: Skips test execution for faster builds
- **Output**: Packaged artifact in `target/` directory

### 7. Deploy to Nexus
```groovy
withMaven(globalMavenSettingsConfig: 'global-settings-xml'){
    sh "mvn deploy -DskipTests=true"
}
```
- **Purpose**: Deploys artifacts to Nexus Repository Manager
- **Configuration**: Uses global Maven settings XML
- **Repositories**: 
  - `maven-releases`: For release versions
  - `maven-snapshots`: For snapshot versions
- **Authentication**: Configured via `global-settings-xml`

### 8. Build & Tag Docker Image
```groovy
withDockerRegistry(credentialsId:'docker-cred', toolName:'docker'){
    sh "docker build -t shopping-cart:dev -f docker/Dockerfile ."
    sh "docker tag shopping-cart:dev hiteshbhalotia/shopping-cart:dev"
}
```
- **Purpose**: Builds Docker image and tags for registry push
- **Dockerfile**: Located at `docker/Dockerfile`
- **Local Tag**: `shopping-cart:dev`
- **Registry Tag**: `hiteshbhalotia/shopping-cart:dev`
- **Credentials**: Uses `docker-cred` for Docker Hub authentication

### 9. Deploy Application
```groovy
withDockerRegistry(credentialsId:'docker-cred', toolName:'docker'){
    sh "docker run -d --name ekart -p 8070:8070 hiteshbhalotia/shopping-cart:dev"
}
```
- **Purpose**: Deploys the application as a Docker container
- **Container Name**: `ekart`
- **Port Mapping**: `8070:8070` (host:container)
- **Mode**: Detached (`-d`) for background execution
- **Access**: Application available at `http://localhost:8070`

## 📁 Expected Project Structure

```
Ekart/
├── src/
│   └── main/
│       ├── java/           # Java source code
│       └── resources/      # Application resources
├── docker/
│   └── Dockerfile         # Docker image build instructions
├── target/                # Maven build output (generated)
├── pom.xml               # Maven project configuration
├── settings.xml          # Maven settings (if local)
└── [other project files]
```

## 🔧 Configuration Requirements

### Maven POM Configuration (pom.xml)
Ensure your `pom.xml` includes Nexus repository configuration:
```xml
<distributionManagement>
    <repository>
        <id>maven-releases</id>
        <url>http://your-nexus-server/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>maven-snapshots</id>
        <url>http://your-nexus-server/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

### Global Settings XML (global-settings-xml)
Configure Maven settings for Nexus authentication:
```xml
<settings>
    <servers>
        <server>
            <id>maven-releases</id>
            <username>${NEXUS_USERNAME}</username>
            <password>${NEXUS_PASSWORD}</password>
        </server>
        <server>
            <id>maven-snapshots</id>
            <username>${NEXUS_USERNAME}</username>
            <password>${NEXUS_PASSWORD}</password>
        </server>
    </servers>
</settings>
```

### Jenkins Credentials Configuration
Set up the following credentials in Jenkins:

1. **Docker Hub Credentials** (`docker-cred`):
   - Type: Username with password
   - Username: Docker Hub username
   - Password: Docker Hub token/password

2. **SonarQube Token** (in SonarQube server configuration):
   - Type: Secret text
   - Value: SonarQube authentication token

## 🛠️ Plugin Installation

Install the required plugins via Jenkins Plugin Manager:

### Core Plugins
- **JDK Tool**: For Java Development Kit management
- **Pipeline Plugin**: For declarative pipeline support
- **Git Plugin**: For source code management

### Security & Quality Plugins
- **OWASP Dependency-Check Plugin**: Vulnerability scanning
- **SonarQube Scanner Plugin**: Code quality analysis

### Build & Deployment Plugins
- **Pipeline Maven Integration Plugin**: Maven pipeline support
- **Config File Provider Plugin**: For settings.xml management
- **Docker Plugin**: Docker integration
- **Docker Pipeline Plugin**: Docker pipeline steps
- **Docker Build Step Plugin**: Enhanced Docker build capabilities

### Installation Steps
1. Navigate to Jenkins → Manage Jenkins → Manage Plugins
2. Go to "Available" tab
3. Search and install each plugin listed above
4. Restart Jenkins when prompted

## 🔧 Jenkins Configuration

### 1. SonarQube Server Configuration
1. Navigate to Jenkins → Manage Jenkins → Configure System
2. Find "SonarQube servers" section
3. Add SonarQube server:
   - **Name**: `sonar`
   - **Server URL**: Your SonarQube server URL
   - **Authentication**: SonarQube token

### 2. Global Tool Configuration
1. Navigate to Jenkins → Manage Jenkins → Global Tool Configuration
2. Configure all tools as specified in the Tool Configurations section

### 3. Config File Provider Setup
1. Navigate to Jenkins → Manage Jenkins → Managed files
2. Add new config file:
   - **Type**: Global Maven settings.xml
   - **ID**: `global-settings-xml`
   - **Content**: Maven settings with Nexus configuration

## 📊 Security Scanning Reports

### OWASP Dependency Check
- **Report Location**: `dependency-check-report.xml`
- **Jenkins Integration**: Published with trend analysis
- **Access**: Build results → Dependency-Check Results
- **Coverage**: All project dependencies and transitive dependencies

### Trivy Scan Results
- **Output**: Jenkins console logs
- **Coverage**: Filesystem, dependencies, and configurations
- **Format**: Human-readable console output

### SonarQube Analysis
- **Integration**: Real-time results in SonarQube dashboard
- **Metrics**: Code coverage, bugs, vulnerabilities, maintainability
- **Quality Gates**: Configurable pass/fail criteria

## 🚀 Artifact Management

### Nexus Repository Integration
The pipeline automatically deploys artifacts to Nexus based on version:

#### Release Artifacts
- **Repository**: `maven-releases`
- **Condition**: When project version doesn't contain "SNAPSHOT"
- **Retention**: Permanent storage

#### Snapshot Artifacts
- **Repository**: `maven-snapshots`
- **Condition**: When project version contains "SNAPSHOT"
- **Retention**: Configurable cleanup policies

### Artifact Types
- **JAR Files**: Primary application artifact
- **Source JARs**: Source code archives (if configured)
- **Javadoc JARs**: Documentation archives (if configured)
- **POM Files**: Project metadata

## 🐳 Docker Deployment

### Image Building Process
1. **Base Image**: Defined in `docker/Dockerfile`
2. **Application Layer**: Adds compiled JAR and dependencies
3. **Configuration**: Environment-specific settings
4. **Tagging**: Version and environment-specific tags

### Container Deployment
- **Port**: Application runs on port 8070
- **Health Check**: Manual verification required
- **Logs**: Available via `docker logs ekart`
- **Scaling**: Single container deployment

### Access Information
- **URL**: http://localhost:8070 OR http://13.204.69.78:8070 (in my case)
- **Container Name**: `ekart`
- **Network**: Default Docker bridge network

## 🔄 Version History

- **v1.0**: Initial comprehensive pipeline
  - Complete CI/CD workflow
  - Security scanning integration
  - Code quality analysis
  - Artifact management
  - Docker deployment

---

**Note**: This pipeline is optimized for the Ekart shopping cart application. Ensure your project structure, Maven configuration, and Docker setup match the requirements outlined in this documentation.
