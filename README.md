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

Upon completion, the final files are stored in the S3 bucket "nextwork-devops-cicd-shanikah", with the presence of a WAR file confirming a successful build.

Real-time visibility into the process is provided by CloudWatch Logs, which display outputs and errors to facilitate quick troubleshooting.

This automated pipeline ensures consistent builds, minimizes manual intervention, and enhances the feedback loop for developers.

### Automated Deployment with AWS CodeDeploy  
AWS CloudFormation was used to launch an EC2 instance along with its networking resources (VPC, Subnet, Route Tables, Internet Gateway, Security Group). To automate the setup, several deployment scripts were created in VSCode:
- install_dependencies.sh – Installs Tomcat and Apache, configuring Apache as a reverse proxy.
- start_server.sh – Ensures both services start automatically and restart on reboot.
- stop_server.sh – Safely stops services to prevent deployment errors.
- appspec.yml – Defines AWS CodeDeploy deployment steps, including file mappings and lifecycle hooks.
- buildspec.yml – Packages deployment files into the build artifact for CodeDeploy.

![image](https://github.com/user-attachments/assets/e30a7819-3c34-45e2-b606-7bd00578627f)

An AWS CodeDeploy application (nextwork-devops-cicd) was created to automate EC2 deployments. A deployment group was set up, and an IAM role was configured to grant CodeDeploy permissions for managing EC2, S3, Auto Scaling, and CloudWatch logs.

To target the correct instance, the role: webserver tag was applied, allowing the CodeDeploy Agent to listen for deployment instructions.

The web application was deployed using the S3 URI of the build artifact stored in the nextwork-devops-cicd bucket. Deployment success was verified by accessing the EC2 instance's Public IPv4 DNS in a browser.

![Screenshot 2025-05-07 160512](https://github.com/user-attachments/assets/f1817f5b-b9b9-446a-803f-3ca0ce97b683)

### Pipeline Automation with AWS CodePipeline
![image](https://github.com/user-attachments/assets/9bae94d5-c77e-42f8-a0ed-cef6bd259d32)
A three-stage AWS CodePipeline was set up to automatically handle software updates.
The pipeline includes:

Source Stage:
- The pipeline monitors a GitHub repository for changes and automatically triggers upon detecting updates.
 ![image](https://github.com/user-attachments/assets/66dbea8f-0da0-4d12-b4ff-38e627c97e45)

Build:
- AWS CodeBuild compiles and packages the application, ensuring the latest version is ready for deployment.
![image](https://github.com/user-attachments/assets/a29c887e-4e74-4052-ac88-c6c0a851c3c6)

Deploy:
- AWS CodeDeploy updates the EC2 instance seamlessly, ensuring zero-downtime deployment.
![image](https://github.com/user-attachments/assets/b18346a7-4947-4f42-b069-750bf73f83de)

Operating in Superseded mode, the pipeline processes only the latest changes, preventing redundant builds. IAM roles were automatically created to provide the required permissions for each service, boosting both security and automation.

To validate the end-to-end workflow, a front-end change to the index.jsp file was pushed to GitHub, triggering the pipeline to detect the update, build the application, deploy it and reflect the changes live, without manual intervention. This automation streamlines the development workflow, reducing deployment time and minimizing human error.

![Screenshot 2025-05-07 164933](https://github.com/user-attachments/assets/4e4b85f0-5e4a-4c26-a744-160e209002c8)


## Key Takeaways
Successfully automated the entire CI/CD workflow using AWS and GitHub

Emphasized security through IAM roles, key-based SSH, and secure token management

Adopted infrastructure as code via CloudFormation for reproducibility

Demonstrated real-time deployment of application changes with zero downtime

## Future Enhancements
Introduce automated testing within the pipeline

Implement blue/green or canary deployments

Add monitoring, alerting, and logging enhancements with CloudWatch and SNS


##### Network Student
##### Nextwork.org 
![image](https://github.com/user-attachments/assets/fece45c6-e4de-44ba-96aa-b74fde4173a6)



