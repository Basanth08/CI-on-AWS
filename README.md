# Continuous Integration on AWS

Welcome to my **Continuous Integration on AWS** project!

In this repository, I show you how I set up and manage continuous integration (CI) pipelines using AWS services.

---

## Project Overview

In this project, I walk you through my process of implementing CI practices on AWS, leveraging cloud-native tools and best practices that I’ve found effective.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Table of Contents](#table-of-contents)
- [Important Notice: AWS CodeCommit Availability](#important-notice-aws-codecommit-availability)
- [Flow of Execution](#flow-of-execution)
- [Setting Up IAM User and Permissions for CodeCommit](#setting-up-iam-user-and-permissions-for-codecommit)
  - [SSH Key Setup for CodeCommit](#ssh-key-setup-for-codecommit)
  - [Verifying SSH Access to CodeCommit](#verifying-ssh-access-to-codecommit)
  - [Preparing to Migrate My Repository](#preparing-to-migrate-my-repository)
- [Setting Up AWS CodeArtifact](#setting-up-aws-codeartifact)
  - [Installing and Configuring the AWS CLI](#installing-and-configuring-the-aws-cli)
  - [Important: Secure Your Credentials in settings.xml](#important-secure-your-credentials-in-settingsxml)
- [Setting Up SonarCloud](#setting-up-sonarcloud)
- [Using AWS Systems Manager Parameter Store](#using-aws-systems-manager-parameter-store)
- [Monitoring and Debugging CodeBuild Builds](#monitoring-and-debugging-codebuild-builds)
- [Configuring Quality Gates and Build Settings](#configuring-quality-gates-and-build-settings)
  - [Creating a Build Project in AWS CodeBuild](#creating-a-build-project-in-aws-codebuild)
- [Attaching Permissions, Notifications, and Creating Pipelines](#attaching-permissions-notifications-and-creating-pipelines)

---

## Important Notice: AWS CodeCommit Availability

As of July 25th, 2024, AWS CodeCommit has stopped onboarding new customers. Here’s what this means for me (and for you if you’re following along):

- If I already have at least one CodeCommit repository in my AWS account or organization, I can continue to create more repositories as needed.
- If I want to use CodeCommit in a new AWS account (especially within an AWS Organization), I need to submit a support case to AWS. In my request, I’ll need to provide justification for using CodeCommit and confirm that my organization was a CodeCommit customer before July 25th, 2024.
- If neither of the above applies, I recommend using alternative Git-based source control solutions such as GitHub, GitLab, or Bitbucket. AWS also provides guidance for migrating existing CodeCommit repositories to these platforms.

If you have any questions or need help with migration, feel free to reach out!

---

## Flow of Execution

Here’s how I execute my CI pipeline setup on AWS, step by step:

### 1. Login to AWS Account
- I start by logging into my AWS account to access all the necessary services.

### 2. CodeCommit
- I create a CodeCommit repository to store my source code.
- I set up an IAM user with the required CodeCommit policy.
- I generate SSH keys locally and exchange them with the IAM user for secure access.
- I migrate my source code from GitHub to the CodeCommit repository and push it.

### 3. CodeArtifact
- I create an IAM user with access to CodeArtifact.
- I install and configure the AWS CLI on my machine.
- I export the authentication token for CodeArtifact.
- I update the `settings.xml` file in my source code’s top-level directory with the necessary details.
- I also update the `pom.xml` file with repository details for dependency management.

### 4. SonarCloud
- I create a SonarCloud account and generate a token for authentication.
- I store SonarCloud details as SSM parameters in AWS.
- I create a build project that integrates with SonarCloud.
- I update the CodeBuild role to access the SSM Parameter Store for secure credentials.

### 5. Notifications
- I set up notifications for build and deployment status using SNS or Slack, so I’m always in the loop.

### 6. Build Project
- I update the `pom.xml` file to include artifact versioning with a timestamp.
- I create variables in SSM Parameter Store for use in the build process.
- I create the build project in AWS CodeBuild.
- I update the CodeBuild role to access SSM Parameter Store as needed.

### 7. Create Pipeline
- I create a pipeline in AWS CodePipeline that connects all the stages:
  - CodeCommit (source)
  - Test code
  - Build
  - Deploy to an S3 bucket

### 8. Test Pipeline
- Finally, I test the pipeline to make sure everything works as expected from code commit to deployment.

---

## Setting Up IAM User and Permissions for CodeCommit

Since I need secure access to AWS CodeCommit, here’s how I set up my IAM user and permissions:

1. **Add a New IAM User:**  
   I start by creating a new IAM user in the AWS console. I give the user a descriptive name (for example, `vprofile-code-ad`) and select "Programmatic access" so I can use access keys for the AWS CLI, SDK, and other tools.

2. **Attach CodeCommit Policies:**  
   Next, I assign the necessary permissions. I choose to attach existing policies directly and search for AWS-managed policies related to CodeCommit, such as:
   - `AWSCodeCommitFullAccess`
   - `AWSCodeCommitPowerUser`
   - `AWSCodeCommitReadOnly`
   I select the policy that matches the level of access I need.

3. **Fine-Tune Permissions (Optional):**  
   If I want more granular control, I can manually specify actions (like list, read, write) and restrict access to specific repositories by providing the ARN (Amazon Resource Name) for the repository, region, and account.

4. **Review and Create:**  
   After reviewing the permissions, I create the user and securely store the access key ID and secret access key for use in my development environment.

By following these steps, I make sure my AWS CodeCommit access is both secure and tailored to my project’s needs.

### SSH Key Setup for CodeCommit

To securely connect to AWS CodeCommit using SSH, here’s what I do:

1. **Generate an SSH Key Pair:**  
   I use the `ssh-keygen` command to generate a new SSH key pair. For example:
   ```sh
   ssh-keygen -t rsa -b 3072 -C "my-codecommit-key"
   ```
   I save the key with a descriptive name, like `vpro-codecommit_rsa`.

2. **Locate the Public Key:**  
   I navigate to my `.ssh` directory and find the public key file (e.g., `vpro-codecommit_rsa.pub`).

3. **Copy the Public Key Contents:**  
   I open the public key file and copy its contents to my clipboard.

4. **Upload the SSH Public Key to IAM:**  
   In the AWS IAM console, I go to the user’s security credentials and upload the SSH public key under the "SSH keys for AWS CodeCommit" section.

5. **Use the SSH Key for CodeCommit Access:**  
   Now, I can use this SSH key to securely authenticate and interact with my CodeCommit repositories from my local machine.

This setup ensures my connections to CodeCommit are both secure and convenient.

### Verifying SSH Access to CodeCommit

After uploading my SSH public key to IAM, I always verify that my setup works. I do this by running:

```
ssh git-codecommit.us-east-2.amazonaws.com
```

If everything is configured correctly, I’ll see a welcome message or confirmation that my SSH connection to AWS CodeCommit is successful. This step ensures I can securely interact with my repositories.

### Preparing to Migrate My Repository

Before migrating my code from GitHub to CodeCommit, I check my current repository’s configuration. I use:

```
cat .git/config
```

This shows me the current remote URL (for example, pointing to GitHub). I’ll need to update this remote to point to my new CodeCommit repository when I’m ready to push my code to AWS.

---

## Setting Up AWS CodeArtifact

To manage my build artifacts, I use AWS CodeArtifact. Here’s how I set it up:

1. I navigate to the CodeArtifact section in the AWS Developer Tools console and click 'Create repository'.
2. I provide a repository name (for example, 'vprofile-maven-repo') and an optional description.
3. If needed, I specify public upstream repositories (like Maven Central) to allow my repository to pull dependencies from official sources.
4. I click 'Next' and, if prompted, create a domain (for example, 'visualpath') to group my repositories.
5. I review the settings and complete the repository creation process.

This setup allows me to securely store, publish, and share software packages used in my CI/CD pipeline.

### Installing and Configuring the AWS CLI

To interact with AWS services from my local machine, I first install the AWS CLI. On Windows, I use Chocolatey:

```sh
choco install awscli -y
```

Once installed, I configure the CLI with my credentials:

```sh
aws configure
```

I enter my AWS Access Key ID, Secret Access Key, default region, and output format. This setup allows me to securely run AWS commands from my terminal.

### Important: Secure Your Credentials in settings.xml

When I configure my Maven `settings.xml` file to connect to AWS CodeArtifact, I make sure **never to hard-code my AWS credentials** directly in the file. Instead, I use environment variables for sensitive information. For example:

```xml
<server>
  <id>codeartifact</id>
  <username>aws</username>
  <password>${env.CODEARTIFACT_AUTH_TOKEN}</password>
</server>
```

This way, my authentication token is pulled from the environment at build time, keeping my credentials safe and out of version control.  
**Always double-check your `settings.xml` before committing to make sure no secrets are exposed!**

---

## Setting Up SonarCloud

To ensure my code is clean and free of bugs or vulnerabilities, I use SonarCloud for static code analysis. Here’s how I set it up:

1. I go to [sonarcloud.io](https://sonarcloud.io) and log in.
2. I connect my code repository by choosing from GitHub, Bitbucket, Azure DevOps, or GitLab.
3. I select the project I want to analyze from my list of repositories.
4. If needed, I can set up a project manually by providing a project key and display name, and choosing whether the project is public or private.

SonarCloud helps me champion quality code in my projects by eliminating bugs and vulnerabilities. It’s free for open-source projects and easy to integrate into my workflow.

---

## Using AWS Systems Manager Parameter Store

To securely manage and inject sensitive configuration values (like tokens and secrets) into my CI/CD pipeline, I use AWS Systems Manager Parameter Store. Here’s how I set it up:

1. I open the AWS Systems Manager console and navigate to "Parameter Store."
2. I click "Create parameter" and give my parameter a descriptive name (for example, `sonartoken`, `HOST`, or `codeartifact-token`).
3. I choose the "Standard" tier for most use cases, and set the type to "String" (or "SecureString" if I want encryption).
4. I enter the value (such as my SonarCloud token or CodeArtifact token) and save the parameter.

Later, I can reference these parameters in my build and deployment scripts, or configure AWS services (like CodeBuild) to retrieve them securely at runtime. This keeps my secrets out of source code and version control, and makes my pipeline more secure and manageable.

---

## Monitoring and Debugging CodeBuild Builds

Once I start a build in AWS CodeBuild, I closely monitor its progress and results:

- I check the build status and phase details to see each step (like SUBMITTED, QUEUED, PROVISIONING, etc.) and ensure they all succeed.
- I use the Build logs, Phase details, and Reports tabs to view detailed output, debug issues, and verify that my CI/CD pipeline is working as expected.
- Environment variables, build details, and resource utilization tabs help me troubleshoot and optimize my builds.

By monitoring these details, I can quickly identify and resolve any issues, ensuring my automated build and deployment process runs smoothly.

---

## Configuring Quality Gates and Build Settings

To enforce code quality and keep my CI/CD pipeline flexible, I:

- Use SonarCloud to set up quality gates. For example, I can configure a quality gate to fail the build if the number of bugs exceeds a certain threshold.
- Associate a specific quality gate with my project in SonarCloud, ensuring that only code meeting my standards passes the pipeline.
- In AWS CodeBuild, I can edit build project settings at any time—such as the source, environment, buildspec, and logs—so I can adapt my pipeline as my project evolves.

These steps help me maintain high code quality and make my CI/CD process robust and maintainable.

### Creating a Build Project in AWS CodeBuild

When I want to automate and monitor my CI/CD builds, I create a new build project in AWS CodeBuild:

1. I enter a project name and (optionally) a description to identify my build project.
2. I choose a managed image (like Ubuntu) for the build environment, and select the appropriate runtime for my application.
3. I configure the service role, either by creating a new one or selecting an existing role with the necessary permissions.
4. I enable CloudWatch logs and specify a log group name to monitor and troubleshoot my builds easily.

These steps help me automate my build process and keep track of build activity and issues in my CI/CD pipeline.

---

## Attaching Permissions, Notifications, and Creating Pipelines

To automate, secure, and monitor my CI/CD workflow, I:

- Attach custom policies (like `vprofile-sonarparametersaccess`) to my CodeBuild service role, so my build project can access required parameters and secrets.
- Start builds by selecting the branch, tag, or commit I want to build from, and clicking **Start build** to trigger the CI/CD process for that code version.
- Set up notifications using Amazon SNS: I create a topic, subscribe (for example, via email), and confirm the subscription to get notified about build and deployment events.
- Create a pipeline in AWS CodePipeline: I enter a pipeline name, set or create the service role, add source, build, and deploy stages, and add extra stages (like Test) as needed for my workflow.

These steps help me keep my CI/CD pipeline automated, secure, and easy to monitor from end to end.

---


