# Automated CI/CD Pipeline with AWS for Java Web Application
![image](https://github.com/user-attachments/assets/5ad24dfe-b2c7-4063-8994-ce73112a309f)
## Project Overview
The goal was to set up a fully automated CI/CD pipeline using AWS services for a Java-based web application. This project addresses the challenges of manual deployment, which is often slow and error-prone, by creating a reliable, secure, and easy-to-manage workflow covering source control to deployment with modern cloud tools.

The pipeline leverages four key AWS services:
- AWS CodePipeline to orchestrate the process
- AWS CodeBuild for automated builds
- AWS CodeDeploy for reliable deployments
- AWS CodeArtifact for secure dependency management

GitHub handles version control, triggering the pipeline on code changes, leading to deployments on an EC2 instance provisioned via CloudFormation. This integration accelerates deployment frequency, reduces human errors, and improves traceability.
 
### Environment Setup
I began by configuring a remote EC2 instance with secure SSH access, restricting the private key permissions to prevent unauthorized use. 
![image](https://github.com/user-attachments/assets/e2106c4e-0e46-46d4-8682-7ece63271389)
Using Visual Studio Code's Remote – SSH extension, I connected seamlessly to the instance:
`ssh -i "shanikah-keypair.pem" ec2-user@44.202.28.117`

![image](https://github.com/user-attachments/assets/4ff35c90-5381-4851-8293-dd4ca76d2f61)

To establish the core environment, Java and Maven were installed on the EC2 instance. The application’s framework was structured using Maven’s archetype feature to standardize the directory layout: 
```
mvn archetype:generate
-DgroupId=com.mycompany.app
-DartifactId=my-app
-DarchetypeArtifactId=maven-archetype-webapp
-DinteractiveMode=false
```
![image](https://github.com/user-attachments/assets/5fa2346f-f8d4-41c1-abd4-c835ae3db323)

#### Setting Up Git and GitHub on an EC2 Instance  
Git was installed and configured on the EC2 instance:  
```
sudo apt-get update
sudo apt-get install git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```  
A GitHub repository was linked using a Personal Access Token (PAT) for secure push and pull operations. This enabled automatic source control integration to trigger pipeline execution on code changes. 

### Secure Dependency Management with AWS CodeArtifact
To securely manage project dependencies, AWS CodeArtifact was configured with the following:

- A custom domain and repository for organizing and isolating packages
- Maven Central designated as the upstream source
![image](https://github.com/user-attachments/assets/f5c7888a-e206-42aa-bd35-1bf6cfdfb109)

- An IAM role assigned to the EC2 instance for secure authentication
![image](https://github.com/user-attachments/assets/8bf569e1-63ad-410a-888d-04ef0c299dac)

Once everything was configured, Maven successfully pulled packages from CodeArtifact using a `settings.xml` file designed for authentication and access.

![image](https://github.com/user-attachments/assets/2ec5b0f0-6a3f-41c9-b941-00aaa57cbc35)

#### Verifying the Connection
After compiling, the CodeArtifact repository was checked, confirming the presence of Maven packages and a successful integration.
![image](https://github.com/user-attachments/assets/1875755e-c0ab-4f9c-abda-18d170812439)

### Continuous Integration with AWS CodeBuild
AWS CodeBuild was integrated with GitHub via an app, removing the need for manual credential management.

![image](https://github.com/user-attachments/assets/5fd99875-97db-41da-8500-d6541932d86c)

The build was driven by `buildspec.yml`, which outlines the setup, tool installation, and code packaging.
![Screenshot 2025-05-07 144842](https://github.com/user-attachments/assets/3ce61412-7a68-43f4-93d8-40e2a90e4872)

Upon completion, the final files are securely stored in the S3 bucket "nextwork-devops-cicd-shanikah," with the presence of a WAR file confirming a successful build. CloudWatch Logs provide real-time visibility, displaying outputs and errors to streamline troubleshooting.

### Automated Deployment with AWS CodeDeploy  
Using CloudFormation, I provisioned an EC2 instance with networking resources. To automate the setup, several deployment scripts were created in VSCode:
- `install_dependencies.sh` installs Tomcat and Apache (configuring Apache as a reverse proxy).
- `start_server.sh`ensures both services start automatically and restart on reboot.
- `stop_server.sh` safely stops services to prevent deployment errors.
- `appspec.yml` defines AWS CodeDeploy deployment steps, including file mappings and lifecycle hooks.
- `buildspec.yml` packages deployment files for CodeDeploy.

![image](https://github.com/user-attachments/assets/e30a7819-3c34-45e2-b606-7bd00578627f)

An AWS CodeDeploy application and deployment group were created, with IAM roles granting necessary permissions. The EC2 instance was tagged webserver for targeted deployment.

The application deployed from the S3 build artifact, verified by accessing the EC2 instance’s Public IPv4 DNS.

![Screenshot 2025-05-07 160512](https://github.com/user-attachments/assets/f1817f5b-b9b9-446a-803f-3ca0ce97b683)

### Pipeline Automation with AWS CodePipeline
![image](https://github.com/user-attachments/assets/9bae94d5-c77e-42f8-a0ed-cef6bd259d32)
AWS CodePipeline helps keep software up to date automatically, improving efficiency and security.
1) The pipeline begins with the Source Stage, which continuously monitors GitHub for changes and triggers the pipeline when updates are detected.
 ![image](https://github.com/user-attachments/assets/66dbea8f-0da0-4d12-b4ff-38e627c97e45)

2) Next, the Build Stage uses AWS CodeBuild to compile and package the application, ensuring the latest version is deployment-ready. Operating in Superseded mode, the pipeline processes only the most recent changes, preventing redundant builds and optimizing resource usage. IAM roles are automatically generated to grant necessary permissions for each service, improving both security and automation.
![image](https://github.com/user-attachments/assets/a29c887e-4e74-4052-ac88-c6c0a851c3c6)

3) Finally, the Deploy Stage utilizes AWS CodeDeploy to update the EC2 instance seamlessly. This ensures a zero-downtime deployment, allowing users to access the latest version of the software without disruptions.
![image](https://github.com/user-attachments/assets/b18346a7-4947-4f42-b069-750bf73f83de)


## Lessons Learned and Challenges Overcome
**

## Final Deliverables and Reflection
This project implemented a fully automated CI/CD pipeline for web application deployment on EC2. By integrating AWS CodePipeline, CodeBuild, CodeDeploy, and CodeArtifact, it ensured seamless updates with zero downtime.

Through this experience, I gained deeper expertise in AWS DevOps tools, transforming manual workflows into automated pipelines that improved deployment speed and software stability. The final implementation ensured that modifying the index.jsp file in GitHub automatically triggered the pipeline, instantly delivering updates while maintaining consistent application performance.

![Screenshot 2025-05-07 164933](https://github.com/user-attachments/assets/4e4b85f0-5e4a-4c26-a744-160e209002c8)

##### Network Student
##### Nextwork.org 
![image](https://github.com/user-attachments/assets/fece45c6-e4de-44ba-96aa-b74fde4173a6)



