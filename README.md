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
Once the key pair was set up and permissions were configured, I connected to the EC2 instance using the command: ssh -i “nextwork keypair.pem” ec2-user@44.202.28.117. This allowed me to interact with the instance remotely using the instance’s public IPv4 DNS address.
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

