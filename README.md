# AWS CI/CD Pipeline Project

Automated CI/CD pipeline for deploying a Java web application using AWS services.

## What This Project Does

Automatically builds and deploys a web application whenever code changes are pushed to GitHub. The pipeline handles compiling code, running tests, creating deployment packages, and deploying to a production server - all without manual intervention.

## Technologies Used

**AWS Services:**
- EC2 (application hosting)
- CodePipeline (workflow orchestration)
- CodeBuild (automated builds)
- CodeDeploy (automated deployments)
- CodeArtifact (package management)
- CodeConnections (Github integration)
- CloudFormation (infrastructure as code)
- S3 (artifact storage)
- IAM (permissions management)

**Development Tools:**
- Git/GitHub (version control)
- Apache Maven (build automation)
- Java/Apache Tomcat (application runtime)

## How It Works

1. Developer pushes code changes to GitHub
2. CodePipeline detects the change automatically
3. CodeBuild compiles the Java application
4. Build artifact (.war file) is stored in S3
5. CodeDeploy deploys the application to EC2
6. Web application is live with updated code

## Project Documentation

This was a 7-day guided DevOps challenge. Full day-to-day documentation:
Day 1- (https://learn.nextwork.org/jorge_hdez_aws/docs/aws-devops-vscode)
Day 2- (https://learn.nextwork.org/jorge_hdez_aws/docs/aws-devops-github)
Day 3- (https://learn.nextwork.org/jorge_hdez_aws/docs/aws-devops-codeartifact-updated)
Day 4- (https://learn.nextwork.org/jorge_hdez_aws/docs/aws-devops-codebuild-updated)
Day 5- (https://learn.nextwork.org/jorge_hdez_aws/docs/aws-devops-codedeploy-updated)
Day 6- (https://learn.nextwork.org/jorge_hdez_aws/docs/aws-devops-cloudformation-updated)
Day 7- (https://learn.nextwork.org/jorge_hdez_aws/docs/aws-devops-codepipeline-updated)

## What I Learned

- Setting up automated CI/CD workflows using AWS services
- Managing IAM roles and permissions across services
- Infrastructure as code with CloudFormation
- Troubleshooting authentication and deployment issues
- Git workflow integration with AWS services
- Remote development with VS Code SSH extension

## Repository Contents

- `buildspec.yml` - CodeBuild build configuration
- `appspec.yml` - CodeDeploy deployment configuration
- `settings.xml` - Maven/CodeArtifact configuration
- `/scripts` - Deployment automation scripts
- `/src` - Application source code
