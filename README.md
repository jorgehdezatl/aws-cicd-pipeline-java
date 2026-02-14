# AWS CI/CD Pipeline for Java Web Application

Fully automated CI/CD pipeline that builds, tests, and deploys a Java web application to production using AWS managed services and Infrastructure as Code.

![CI/CD Pipeline Architecture](images/AWS%20CI_CD%20Pipeline.png)

---

## 🎯 Project Overview

**Problem:**  
Manual build and deployment processes were time-consuming (2+ hours per deployment), error-prone, and inconsistent across environments. Developers had to manually compile code, run tests, package artifacts, and deploy to servers—increasing the risk of configuration drift and human error.

**Solution:**  
Designed and deployed a fully automated CI/CD pipeline using AWS CodePipeline, CodeBuild, and CodeDeploy with Infrastructure as Code (CloudFormation). The pipeline automatically detects code changes in GitHub, builds and tests the application, stores artifacts, and deploys to production—reducing deployment time from 2+ hours to 15 minutes while eliminating manual errors.

**Impact:**  
- **15-minute deployment time** (vs. 2+ hours manual process)
- **Zero manual build steps** - fully automated workflow from code push to production
- **Consistent deployments** - Infrastructure as Code ensures reproducible environments
- **Automated rollback capability** - CodeDeploy handles failed deployments automatically
- **Audit logging** - Full visibility into build/deployment history

---

## 🛠️ Technologies Used

**AWS Services:**
- **CodePipeline** - Orchestrates the entire CI/CD workflow
- **CodeBuild** - Compiles Java application, runs tests, creates deployment packages
- **CodeDeploy** - Automates deployment to EC2 instances with lifecycle hooks
- **CodeArtifact** - Private Maven repository for dependency management
- **CloudFormation** - Infrastructure as Code for reproducible deployments
- **EC2** - Application hosting (Amazon Linux 2, Apache Tomcat)
- **S3** - Build artifact storage
- **IAM** - Service permissions and access control
- **CodeConnections** - GitHub integration for automated triggers

**Development Tools:**
- Git/GitHub (version control)
- Apache Maven (build automation)
- Java / Apache Tomcat (application runtime)
- VS Code (remote development via SSH)

---

## 🏗️ Architecture

![CI/CD Pipeline Architecture](images/AWS%20CI_CD%20Pipeline.png)

**Pipeline Flow:**
1. Developer pushes code changes to GitHub repository
2. CodePipeline detects change via CodeConnections webhook (automated trigger)
3. **Source Stage:** CodePipeline pulls latest code from GitHub
4. **Build Stage:** CodeBuild compiles Java application using Maven, pulls dependencies from CodeArtifact, runs tests
5. Build artifact (.war file) stored in S3 bucket
6. **Deploy Stage:** CodeDeploy retrieves artifact from S3 and deploys to EC2 instance
7. Application live at production URL

**Infrastructure Management:**
- CloudFormation provisions all pipeline resources (EC2, CodePipeline, CodeBuild, CodeDeploy, S3, IAM roles)
- IAM manages cross-service permissions (CodeBuild → CodeArtifact, CodeDeploy → EC2, etc.)

---

## 📋 Implementation Journey

### Phase 1: Development Environment Setup
**Goal:** Configure remote development environment and initialize web application

- Set up EC2 instance (Amazon Linux 2) with Java, Maven, and Apache Tomcat
- Configured VS Code SSH remote development for secure connection to EC2
- Initialized Java web application using Maven project structure
- Tested local build and Tomcat deployment

**Key Learning:** VS Code remote SSH allows full IDE experience on cloud instances—eliminating the need for local development environment setup.

---

### Phase 2: Version Control Integration
**Goal:** Establish Git workflow and GitHub integration

- Initialized Git repository on EC2 instance
- Created GitHub repository for code storage and collaboration
- Configured Git commit workflow and pushed initial codebase
- Set up `.gitignore` for build artifacts and IDE files

**Challenge Solved:** Learned Git command workflow (`git add`, `git commit`, `git push`) and resolved SSH authentication for GitHub access.

---

### Phase 3: Package Repository with CodeArtifact
**Goal:** Centralize dependency management with private Maven repository

- Created CodeArtifact domain and repository for Maven packages
- Configured upstream repository (Maven Central) for automatic package caching
- Updated `settings.xml` with CodeArtifact authentication token for Maven builds
- Configured IAM role for EC2 → CodeArtifact access

**Challenge Solved:** Debugged IAM permission errors by creating proper policies for CodeArtifact token generation. Resolved authentication token expiration by implementing token refresh in build scripts.

**Impact:** Centralized package management ensures consistent dependency versions across all builds. Upstream repository caches packages locally, improving build speed and reducing external dependencies.

---

### Phase 4: Automated Builds with CodeBuild
**Goal:** Automate compilation, testing, and artifact packaging

- Created `buildspec.yml` defining build phases (install, pre_build, build, post_build)
- Configured CodeBuild project with Maven build environment
- Integrated CodeArtifact for dependency resolution during builds
- Set up S3 bucket for build artifact storage (.war files)

**Challenge Solved:** Troubleshot CodeArtifact authentication errors in buildspec by properly configuring `CODEARTIFACT_AUTH_TOKEN` environment variable. Debugged Maven dependency resolution issues by verifying `settings.xml` repository URLs.

**Impact:** Automated builds ensure every code change is compiled and tested consistently. Build artifacts stored in S3 create an audit trail and enable rollback to previous versions.

---

### Phase 5: Automated Deployment with CodeDeploy
**Goal:** Automate application deployment to EC2 with zero-downtime releases

- Created `appspec.yml` defining deployment lifecycle hooks (ApplicationStop, BeforeInstall, AfterInstall, ApplicationStart)
- Wrote deployment scripts (`install_dependencies.sh`, `start_server.sh`, `stop_server.sh`)
- Configured CodeDeploy application and deployment group targeting EC2 instance
- Installed and configured CodeDeploy agent on EC2

**Challenge Solved:** 
- **First deployment failure:** Missing IAM permissions for CodeDeploy → S3 access. Created proper IAM role with S3 read permissions.
- **Second deployment failure:** Deployed outdated artifact (before `appspec.yml` was added). Re-ran CodeBuild to generate updated artifact with deployment scripts included.
- **Script permissions:** Ensured deployment scripts had executable permissions (`chmod +x`) in repository.

**Impact:** Automated deployments eliminate manual SSH sessions and file transfers. Lifecycle hooks enable graceful application restarts with proper cleanup and health checks.

---

### Phase 6: Infrastructure as Code with CloudFormation
**Goal:** Convert manual infrastructure into reproducible CloudFormation templates

- Used IaC Generator to scan existing AWS resources and create base template
- Refined template to include EC2 instance, CodePipeline, CodeBuild, CodeDeploy, S3 bucket, and IAM roles/policies
- Deployed infrastructure stack via CloudFormation

**Challenge Solved:**
- **Circular dependency error:** IAM policies referenced roles that didn't exist yet, and roles referenced policies. Fixed by removing `ManagedPolicyArns` from role definitions and using inline policies instead.
- **Resource creation order:** Added `DependsOn` attributes to ensure IAM roles created before policies referencing them.

**Impact:** Infrastructure as Code enables:
- Reproducible environments (dev, staging, prod) from single template
- Version control for infrastructure changes
- Fast disaster recovery (rebuild entire stack in minutes)
- Easy sharing across teams

---

### Phase 7: Full Pipeline Orchestration with CodePipeline
**Goal:** Tie all services together into fully automated workflow

- Created CodePipeline with three stages: Source (GitHub), Build (CodeBuild), Deploy (CodeDeploy)
- Configured CodeConnections for GitHub webhook integration (automatic pipeline triggers on code push)
- Set up IAM roles for cross-service permissions (Pipeline → Build → Deploy)
- Tested end-to-end workflow: code push → automatic build → automatic deployment

**Challenge Solved:**
- **Rebuilt infrastructure from scratch** without step-by-step guidance to reinforce learning
- Troubleshot SSH key permissions for EC2 access
- Fixed CodeArtifact repository name mismatch in `settings.xml`
- Debugged IAM permission issues between pipeline stages

**Impact:** True CI/CD achieved—developers push code, pipeline handles everything automatically. No manual steps, consistent deployments, full audit trail.

---

## 🔧 Key Configuration Files

### `buildspec.yml` - CodeBuild Configuration
```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      java: corretto17
      
  pre_build:
    commands:
      - echo Logging in to CodeArtifact...
      - export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token --domain your-domain --domain-owner your-account-id --query authorizationToken --output text`
      
  build:
    commands:
      - echo Build started on `date`
      - mvn clean package
      
  post_build:
    commands:
      - echo Build completed on `date`

artifacts:
  files:
    - target/*.war
  name: BuildArtifact
```

### `appspec.yml` - CodeDeploy Configuration
```yaml
version: 0.0
os: linux

files:
  - source: /
    destination: /home/ec2-user/server

hooks:
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 300
      
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
```

### `settings.xml` - Maven/CodeArtifact Configuration
```xml
<settings>
  <servers>
    <server>
      <id>codeartifact</id>
      <username>aws</username>
      <password>${env.CODEARTIFACT_AUTH_TOKEN}</password>
    </server>
  </servers>
  
  <profiles>
    <profile>
      <id>default</id>
      <repositories>
        <repository>
          <id>codeartifact</id>
          <url>https://your-domain-your-account-id.d.codeartifact.region.amazonaws.com/maven/your-repo/</url>
        </repository>
      </repositories>
    </profile>
  </profiles>
</settings>
```

---

## 💡 Key Learnings

### Technical Skills Developed
- **CI/CD Pipeline Design:** Architected multi-stage automated workflow integrating source control, build automation, and deployment
- **Infrastructure as Code:** Converted manual infrastructure into CloudFormation templates for reproducible, version-controlled environments
- **AWS Service Integration:** Configured cross-service IAM permissions and service-to-service communication (CodeBuild ↔ CodeArtifact, CodeDeploy ↔ S3)
- **Build Automation:** Implemented Maven builds with dependency management via private artifact repository
- **Deployment Automation:** Created deployment scripts with lifecycle hooks for zero-downtime releases

### Problem-Solving Experience
- **IAM Permission Debugging:** Systematically resolved authentication errors by tracing service-to-service access requirements
- **CloudFormation Troubleshooting:** Fixed circular dependency errors and resource creation ordering issues in IaC templates
- **Build Failures:** Diagnosed and resolved Maven dependency resolution problems and CodeArtifact token configuration
- **Deployment Failures:** Debugged deployment script permissions, artifact versioning, and application lifecycle issues

### DevOps Best Practices
- **Automation First:** Eliminate manual steps to reduce errors and improve consistency
- **Immutable Artifacts:** Store build outputs in S3 for audit trail and rollback capability
- **Infrastructure as Code:** Version control infrastructure alongside application code
- **Centralized Dependencies:** Use artifact repositories for consistent package versions across teams

---

## 🚀 Running This Project

### Prerequisites
- AWS Account with appropriate permissions
- GitHub account
- Basic knowledge of Java, Maven, and Git

### Deployment Steps

1. **Clone Repository**
```bash
   git clone https://github.com/jorgehdezatl/aws-cicd-pipeline-java.git
   cd aws-cicd-pipeline-java
```

2. **Update CloudFormation Template**
   - Edit `cloudformation.yml` with your AWS account details
   - Update region, repository names, and resource names as needed

3. **Deploy Infrastructure Stack**
```bash
   aws cloudformation create-stack \
     --stack-name cicd-pipeline \
     --template-body file://cloudformation.yml \
     --capabilities CAPABILITY_NAMED_IAM
```

4. **Configure CodeConnections**
   - In AWS Console, approve GitHub connection for CodePipeline
   - Authorize repository access

5. **Trigger Pipeline**
   - Push code changes to GitHub repository
   - CodePipeline automatically detects change and runs full workflow

---

## 📁 Repository Structure
```
aws-cicd-pipeline-java/
├── images/
│   ├── cicd-architecture.png
│   ├── pipeline-success.png
│   ├── codebuild-logs.png
│   └── deployed-app.png
├── scripts/
│   ├── install_dependencies.sh
│   ├── start_server.sh
│   └── stop_server.sh
├── src/
│   └── main/
│       └── webapp/
│           └── index.jsp
├── buildspec.yml
├── appspec.yml
├── settings.xml
├── cloudformation.yml
├── pom.xml
└── README.md
```

---

## 🔗 Related Projects

- [Self-Healing AWS Infrastructure with Terraform](https://github.com/jorgehdezatl/self-healing-app-terraform)
- [Three-Tier Serverless Web Application](https://github.com/jorgehdezatl/three-tier-serverless-app)

---

## 📝 Project Timeline

**Total Duration:** 7 days (approx. 12-15 hours)

- Day 1: Development environment setup (1 hour)
- Day 2: Git/GitHub integration (1.5 hours)
- Day 3: CodeArtifact configuration (1.5 hours)
- Day 4: CodeBuild automation (2 hours)
- Day 5: CodeDeploy setup (2 hours)
- Day 6: CloudFormation IaC (2 hours)
- Day 7: CodePipeline orchestration (3 hours)