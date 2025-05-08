# Project Overview
In this project, I’m building and implementing a Continuous Integration and Continuous Delivery (CI/CD) pipeline using AWS services. The process involves setting up a cloud-based web application, integrating it with a GitHub repository, managing dependencies via CodeArtifact, packaging the application with CodeBuild, deploying it through CodeDeploy, and automating the entire workflow with CodePipeline.
![image](https://github.com/user-attachments/assets/5ad24dfe-b2c7-4063-8994-ce73112a309f)
 
## DevOps: Automating Software Delivery
DevOps is a method that encourages teamwork between development and operations teams to enhance and accelerate software delivery through automation and consistency. A DevOps engineer oversees automated workflows, packages applications, and manages cloud systems, emphasizing CI/CD (Continuous Integration and Continuous Deployment). CI/CD simplifies development by frequently merging code, automating builds and tests, and ensuring code is always prepared for deployment, with Continuous Deployment automatically launching changes after they pass testing.

## Setting Up a Web Application Using AWS Services and VSCode.
To start, I installed Visual Studio Code (VSCode), a lightweight yet powerful code editor that is highly customizable. VSCode’s rich ecosystem of extensions, built-in support for remote development, and seamless Git integration made it the perfect tool for managing the development of the web application, regardless of the developer's skill level.

With VSCode installed, I connected to the EC2 instance securely using the Remote – SSH extension. This enabled me to access the project files on the remote server and navigate the directory structure just as if I were working locally, ensuring a smooth and efficient development process.

For managing dependencies and automating the build process, the project was set up with Maven. This streamlined the workflow by handling tasks like compiling code, managing libraries, and ensuring consistent builds across environments.

# Launching an EC2 Instance
To kick off this project, I deployed an Elastic Compute Cloud (EC2) instance, which provides scalable and secure cloud computing resources. As an AWS service, EC2 integrates seamlessly with other AWS services, offering flexibility for various workloads while ensuring secure access and resource management.

## Enabling SSH for Secure Access
To securely access the EC2 instance, I enabled SSH (Secure Shell), allowing for remote administrative tasks while ensuring secure communication with the server.

## Setting Up Key Pairs
Key pairs are used for secure authentication when accessing AWS instances. Each pair consists of:
•	Public Key: Stored on the EC2 instance.
•	Private Key: Retained by the user for secure access.
AWS automatically generated the private key file upon creating the key pair, which is crucial for accessing the EC2 instance. Since this file cannot be recovered if lost, it's important to store it securely. Furthermore, to ensure the private key is secure, I updated its permissions using the following command:
![image](https://github.com/user-attachments/assets/e2106c4e-0e46-46d4-8682-7ece63271389)

This command restricts the key file to read-only access for the owner, preventing unauthorized access.

## SSH connection to EC2 instance 
Once the key pair was set up and permissions were configured, I connected to the EC2 instance using the command: `ssh -i “nextwork keypair.pem” ec2-user@44.202.28.117`. This allowed me to interact with the instance remotely using the instance’s public IPv4 DNS address.
![image](https://github.com/user-attachments/assets/4ff35c90-5381-4851-8293-dd4ca76d2f61)

# Maven & Java 
Apache Maven is a build automation tool that simplifies dependency management, code compilation, testing, and packaging for Java projects. It is crucial for this project as it automates the build process and ensures consistency across development environments. Java is a high-level programming language known for its portability, object-oriented design, and robust performance. It’s used extensively in web, mobile, and enterprise applications, and is well-suited for this project due to its cross-platform capabilities and extensive libraries.

## Create the Application 
To generate the Java web application, I ran the following Maven command:
`mvn archetype: generate` 
`-DgroupId=com.mycompany.app`
`-DartifactId=my-app` 
`-DarchetypeArtifactId=maven-archetype-webapp`
`-DinteractiveMode=false`
This sets up the necessary project structure and dependencies.
![image](https://github.com/user-attachments/assets/5fa2346f-f8d4-41c1-abd4-c835ae3db323)

## Navigating ad Customizing a Web Application in VSCode
In VSCode's file explorer, I navigated the project directory. The src folder contained Java application logic, while the webapp folder held web resources like HTML, JSP files, and static assets.

The index.jsp file acted as the main entry point. Using VSCode, I edited index.jsp to adjust the front-end's message.

![image](https://github.com/user-attachments/assets/7cc384d7-5d2d-4273-abb2-4ccd46e3f133)

# Connect a GitHub Repo with AWS
## Streamlining Version Control and Collaboration 
GitHub is a platform for version control and collaboration built on Git, enabling multiple developers to work on a project simultaneously, track changes, and manage code versions. In this project, I created a remote GitHub repository, generated a Personal Access Token (PAT) for authentication, and pushed local changes to keep the project files backed up and accessible.

To set up version control, I installed Git on my EC2 instance using the commands sudo apt-get update and sudo apt-get install git, allowing me to track changes locally, while GitHub acted as the remote repository for collaboration and version management.

## Setting Up the Local Repository
To initialize a local Git repository, I ran the following command in the web app folder: `git init`
This command set up version control, enabling me to track changes and manage different versions of the project.

## Pushing Changes to GitHub
To push local changes to GitHub, I followed these steps:
1.	Add Changes:
First, I staged the modified files with: `git add .`
2.	Commit Changes:
Then, I committed the changes with a descriptive message: `git commit -m "Your commit message"`
3.	Push Changes:
Finally, I pushed the commit to the remote repository: `git push -u origin master`. The -u flag set the upstream branch for future pushes.

## Authentication and Identity Setup
GitHub requires authentication for committing changes. Since password authentication is no longer supported, I used a Personal Access Token (PAT). To set up a PAT:
•	Navigate to Settings > Developer Settings > Personal Access Tokens.
•	Click Generate New Token, configure the necessary settings, and copy the generated token.
Additionally, I configured my local Git identity:
`git config --global user.name "Your Name"`
`git config --global user.email "your.email@example.com"`
This ensures that all commits are properly attributed.

## Tracking Changes with Git
To view the commit history, I used: `git log`
This command shows detailed commit information, including commit hashes, authors, dates, and messages.

# Secure Packages with CodeArtifact
In this section of the project, this project demonstrates how to set up AWS CodeArtifact to secure the packages used in my web app. The goal is to learn how CodeArtifact reduces errors and risks by ensuring safe and verified package usage. The setup involves configuring IAM, verifying access, and uploading packages.

## Key Tools and Concepts
The primary services used were Amazon EC2 and AWS CodeArtifact. I gained insights into:
•	Setting up artifact repositories
•	Managing Java dependencies
•	Implementing IAM access control for repositories
•	Integrating build processes with dependency management

## CodeArtifact Repository 
CodeArtifact serves as a secure, centralized repository for storing software packages. It helps engineering teams improve security, ensure reliable access to dependencies, and maintain consistent package versions across projects.
A domain groups multiple repositories for easier management of permissions and security. My domain, called "nextwork," simplifies access and control.
CodeArtifact repositories can use upstream repositories to fetch missing packages. My repository uses Maven Central as an upstream source to access and cache Java libraries, ensuring faster and more reliable builds.
![image](https://github.com/user-attachments/assets/f5c7888a-e206-42aa-bd35-1bf6cfdfb109)

## CodeArtifact Security
### Issue:
To access CodeArtifact, an authorization token is required for Maven to retrieve packages. I encountered an error because my EC2 instance lacked the necessary permissions.

### Resolution:
To fix this, I attached an IAM role to my EC2 instance and re-ran the token export command. This worked because the IAM role provided the required permissions for accessing CodeArtifact. Using IAM roles is a security best practice, as they grant only the necessary permissions, minimizing the risk of over-permissioning.

### IAM Policy:
The JSON policy attached to my role grants the necessary permissions to authenticate, access, and read packages from CodeArtifact, enabling Maven to interact with the repository during the build process.
![image](https://github.com/user-attachments/assets/8bf569e1-63ad-410a-888d-04ef0c299dac)

# Maven and CodeArtifact Integration
To test the connection between Maven and CodeArtifact, I compiled my web app using settings.xml. This file configures Maven to authenticate with the CodeArtifact repository and use it as a backup source for dependencies.

Compiling the app translates the code into a computer-understandable format, ensuring everything is ready for deployment.
![image](https://github.com/user-attachments/assets/2ec5b0f0-6a3f-41c9-b941-00aaa57cbc35)

## Verifying the Connection
After compiling, I checked the CodeArtifact repository and confirmed that Maven packages were listed, indicating a successful connection.
![image](https://github.com/user-attachments/assets/1875755e-c0ab-4f9c-abda-18d170812439)

# Continuous Integration with CodeBuild
## Setting Up CI with CodeBuild
In this section, I’ll demonstrate setting up Continuous Integration (CI) using AWS CodeBuild for automated web app builds, GitHub integration, buildspec.yml configuration, and automated testing. The goal of this project is to learn practical CI techniques for faster, more reliable software delivery.

## Key Tools and Concepts
The primary services used include AWS CodeBuild (and potentially CodeArtifact and S3). Key concepts explored are:
•	Continuous Integration (CI)
•	Build automation
•	buildspec.yml
•	GitHub integration
•	CI pipeline role and associated IAM permissions

## Setting Up a CodeBuild Project
CodeBuild is a Continuous Integration service that automates code building and testing, helping teams catch issues early and maintain code quality without managing infrastructure. My project is configured to pull code from GitHub to automate builds directly from my existing repository.

## Connecting CodeBuild with GitHub
To connect CodeBuild with GitHub, I used GitHub App authentication, which is the most secure and straightforward method. This eliminates the need to manage tokens or keys manually, as AWS handles the connection through AWS CodeConnections using the AWS Connector for GitHub.
![image](https://github.com/user-attachments/assets/5fd99875-97db-41da-8500-d6541932d86c)

# CodeBuild Configurations
## Environment:
The CodeBuild project is configured to use an Amazon Linux environment with Corretto 8, on-demand provisioning, EC2 compute, and a new service role for permissions.

## Artifacts:
The build artifacts (such as WAR files) are the output of the build and are essential for deployment. My project creates a WAR file, which is stored in an S3 bucket named nextwork-devops-cicd-yourname.

## Packaging:
I chose to package build artifacts in a Zip file to reduce file size, speed up uploads, and improve storage efficiency. This also simplifies deployment and artifact management.

## Monitoring
For monitoring, I enabled CloudWatch Logs to track and record the build process, including commands, outputs, and errors. This is invaluable for debugging and understanding build issues.

## buildspec.yml
The first build failed because CodeBuild couldn't locate the buildspec.yml file, which defines the build process. The buildspec.yml contains four key phases:
1.	install: Sets up Corretto 8.
2.	pre_build: Retrieves the CodeArtifact token.
3.	build: Compiles the project using Maven.
4.	post_build: Packages the project into a WAR file.
This structure ensures a clear, organized build process.
![image](https://github.com/user-attachments/assets/a595003a-cbe0-4499-8e4d-f45036f29c2e)

# Fixing Build Errors
The second build failed due to missing artifact paths. The solution was to grant the CodeBuild IAM role the necessary permissions to access CodeArtifact for dependencies. I resolved this by attaching the codeartifact-nextwork-consumer-policy to the CodeBuild role. After retrying the build, it succeeded.

# Verifying the Build
To verify the success of the build, I checked the S3 bucket. The presence of the WAR file confirmed that the build was successful: the code was compiled, packaged, and uploaded correctly for deployment.
![image](https://github.com/user-attachments/assets/3fb0b249-38ca-44d1-a0e1-faecbfd9caa3)

# Deploy a Web App with CodeDeploy
## Automating Web App Deployment with CodeDeploy
In this section, I’ll demonstrate how to automate web app deployment to EC2 using AWS CodeDeploy. The focus is on learning continuous deployment, configuring deployment scripts, and implementing rollbacks for efficient, reliable releases, completing the CI/CD pipeline.

## Key Tools and Concepts
The primary services used are AWS CodeDeploy, CloudFormation, EC2, and IAM. Key concepts include:
•	Continuous Deployment
•	Deployment automation
•	Deployment scripts
•	EC2 deployment management
•	Infrastructure as code (CloudFormation)
•	Deployment rollbacks

## Deployment Environment
To set up the environment for CodeDeploy, I launched an EC2 instance and VPC for the necessary compute power and secure networking. Instead of manually provisioning these resources, I used CloudFormation, which allows me to easily delete the resources by removing the CloudFormation stack.

The CloudFormation template creates networking resources such as the VPC, Subnets, Route Tables, Internet Gateway, and Security Group to provide a secure and connected environment for the EC2 instance hosting the web app.
![image](https://github.com/user-attachments/assets/7c8da34c-ba9d-43c6-8993-29ad65bd2df6)

## Deployment Scripts
Deployment scripts automate tasks to configure the EC2 instance for deployment. The scripts used in this project include:
•	install_dependencies.sh: Installs Tomcat and Apache, and configures Apache as a reverse proxy to Tomcat, ensuring the app is accessible on the web.
•	start_server.sh: Starts Tomcat and Apache, and configures them to restart automatically if the EC2 instance reboots, ensuring the app remains running.
•	stop_server.sh: Checks if Apache and Tomcat are running, and if so, stops them safely to ensure a smooth deployment.

## appspec.yml
The appspec.yml file defines the deployment process for CodeDeploy. It includes key sections like:
•	version: Specifies the version of the deployment configuration.
•	os: Defines the operating system for the deployment.
•	files: Maps the artifact to the server.
•	hooks: Specifies the deployment lifecycle scripts.
Additionally, I updated the buildspec.yml file to include the appspec.yml and scripts folder in the artifacts section, ensuring CodeDeploy has the necessary files for deployment.
![image](https://github.com/user-attachments/assets/e30a7819-3c34-45e2-b606-7bd00578627f)

## Setting Up CodeDeploy
A deployment group is a collection of EC2 instances where the app will be deployed. A CodeDeploy application is the container for the deployment group and its associated settings.

To set up the deployment group, I created an IAM role to grant CodeDeploy the permissions needed to access and manage EC2 instances, S3 buckets, Auto Scaling, and CloudWatch logs during the deployment. Tags are used to help CodeDeploy target specific EC2 instances, and I used the role: webserver tag to select the correct instance.
![image](https://github.com/user-attachments/assets/ef2b3d82-4963-4a6d-8e47-4a39a643f435)

## Deployment Configurations
The deployment configuration determines the speed of deployment. I used CodeDeployDefault.AllAtOnce, meaning the app deploys to all instances simultaneously. The CodeDeploy Agent on the EC2 instance communicates with CodeDeploy, executing deployment instructions from appspec.yml. I updated the agent every 14 days to keep it current.
![image](https://github.com/user-attachments/assets/071187cc-745b-480a-aa42-27f1df968634)

# Success!
A CodeDeploy deployment updates the application on EC2 instances. The key distinction between a deployment group and a deployment is that the group refers to the collection of instances, while the deployment is the process of updating them.

I specified a revision location to indicate where CodeDeploy could find the build artifact. This was the S3 URI of the build artifact stored in the nextwork-devops-cicd bucket.

To verify the deployment, I accessed the EC2 instance’s Public IPv4 DNS in a browser and confirmed that the web application was running.

## Building a CI/CD Pipeline with AWS CodePipeline
In this section, I’m setting up a CI/CD pipeline using AWS CodePipeline to automate software builds, tests, and deployments, ensuring efficient, reliable, and continuous delivery.

## Key Tools and Concepts
The services used include CodePipeline, CodeBuild, CodeDeploy, S3, EC2, IAM, and CloudFormation. Key concepts covered are:
•	Full CI/CD automation
•	Integrated build/test/deploy workflow
•	Pipeline stages and artifact flow
•	Automated software delivery
•	Infrastructure as Code (IaC)

## Starting a CI/CD Pipeline
AWS CodePipeline is a managed service that automates software release workflows, facilitating CI/CD by moving code through source, build, and deployment stages. It ensures consistent, automated updates with minimal human intervention.

CodePipeline offers three execution modes: Superseded (cancels old runs for new ones), Queued (runs sequentially), and Parallel (runs concurrently). I selected Superseded to prioritize processing the latest code.

During setup, CodePipeline automatically creates a service role, enabling access to resources like S3 for artifacts and CodeBuild for builds, without requiring manual permission configurations.

## CI/CD Stages
My CI/CD pipeline consists of three stages: Source, Build, and Deploy. The stages are as follows:
•	Source Stage: Retrieves code from GitHub, triggering the pipeline when code changes occur. I enabled webhook events to automatically trigger the pipeline on code pushes.
•	Build Stage: Uses CodeBuild to compile and package the application, with input from the SourceArtifact (the zipped code from the Source stage).
•	Deploy Stage: Deploys the BuildArtifact to the application using CodeDeploy, with automatic rollback enabled in case of errors.
CodePipeline shows the stages with color-coded statuses, and detailed logs can be accessed by clicking on a stage in the "Executions" tab.
![image](https://github.com/user-attachments/assets/9bae94d5-c77e-42f8-a0ed-cef6bd259d32)

# Source Stage
The Source stage monitors the default branch (set to master) for code changes, which trigger the pipeline. By enabling webhook events, I ensure that changes pushed to GitHub automatically trigger the pipeline, providing real-time updates and minimizing manual intervention.
![image](https://github.com/user-attachments/assets/66dbea8f-0da0-4d12-b4ff-38e627c97e45)

# Build Stage
The Build stage compiles and packages the application using CodeBuild, with the source code from the Source stage as input. The result is a BuildArtifact ready for deployment.
![image](https://github.com/user-attachments/assets/a29c887e-4e74-4052-ac88-c6c0a851c3c6)

# Deploy Stage
The Deploy stage uses CodeDeploy to deploy the BuildArtifact to the EC2 instance and deployment group. Automatic rollback is enabled to ensure deployment integrity.
![image](https://github.com/user-attachments/assets/b18346a7-4947-4f42-b069-750bf73f83de)

# Success!
To test the pipeline, I updated the index.jsp file, added a new line, and pushed the change to GitHub. CodePipeline detected the change, triggered the build, and deployed the update automatically.

I verified the pipeline’s success by checking the live web app via the EC2 instance’s Public IPv4 DNS. The updated content was visible, confirming the successful execution of the pipeline.
![image](https://github.com/user-attachments/assets/fcdaacb1-d796-4910-b6ac-e4754b801464)

# Conclusion
This project successfully implemented a fully automated CI/CD pipeline on AWS, demonstrating the practical application of DevOps principles. By integrating GitHub, CodeArtifact, CodeBuild, CodeDeploy, and CodePipeline, the process of building, testing, and deploying the web application was streamlined and automated, leading to faster and more reliable software delivery. This automation reduces manual intervention, minimizes errors, and ensures consistency across the development lifecycle. Further steps could involve adding automated testing stages within the pipeline, exploring different deployment strategies, and implementing more sophisticated monitoring and logging. This experience provided valuable insights into building and managing modern software delivery pipelines in the cloud.

