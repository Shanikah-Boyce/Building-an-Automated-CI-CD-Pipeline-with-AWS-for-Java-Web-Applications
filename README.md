# Automated CI/CD Pipeline with AWS for Java Web Application
![image](https://github.com/user-attachments/assets/5ad24dfe-b2c7-4063-8994-ce73112a309f)
## Project Overview
In this project, the goal was to set up a fully automated CI/CD pipeline using AWS services for a Java-based web application. The primary goal was to create a reliable, secure and easy-to-manage workflow, covering everything from source control to deployment with modern cloud tools.

The pipeline was built using four essential AWS services:

- AWS CodePipeline to orchestrate the entire process 
- AWS CodeBuild for efficient and automated builds  
- AWS CodeDeploy for smooth and reliable deployments
- AWS CodeArtifact for secure managemnt of dependencies 

GitHub was used for version control, ensuring that any changes automatically triggered the pipeline, leading to deployments on an EC2 instance provisioned through CloudFormation. This made software updates faster and more efficient.
 
### Environment Setup
The development process started by configuring a remote EC2 instance and setting up secure SSH access with a key pair. I secured the private key by setting its permissions to read-only for the owner, preventing unauthorized access.
![image](https://github.com/user-attachments/assets/e2106c4e-0e46-46d4-8682-7ece63271389)

Using the "Remote – SSH extension" in Visual Studio Code, remote development felt seamless. Once the key pair was ready, I connected to the EC2 instance with `ssh -i "shanikah-keypair.pem" ec2-user@44.202.28.117`, enabling interaction via its public IPv4 DNS address.

![image](https://github.com/user-attachments/assets/4ff35c90-5381-4851-8293-dd4ca76d2f61)

To establish the core environment, Java and Maven were installed on the EC2 instance. The application’s framework was then structured using Maven’s archetype feature, standardized the directory layout for code and web assets. This was achieved with the command:  

```
mvn archetype:generate
-DgroupId=com.mycompany.app
-DartifactId=my-app
-DarchetypeArtifactId=maven-archetype-webapp
-DinteractiveMode=false
```
![image](https://github.com/user-attachments/assets/5fa2346f-f8d4-41c1-abd4-c835ae3db323)

#### Setting Up Git and GitHub on an EC2 Instance  
Git was installed on the EC2 instance using:  
```
sudo apt-get update
sudo apt-get install git
```  
Next, Git was configured to track commits by setting the user identity:  
```
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```  
Finally, a GitHub repository was linked to the EC2 instance for version control and collaboration, with a Personal Access Token (PAT) ensuring secure push and pull operations.  

### Secure Packages with AWS CodeArtifact
To securely manage project dependencies, AWS CodeArtifact was configured with the following setup:

- A custom domain and repository for organizing and isolating packages
- Maven Central set as the upstream source
![image](https://github.com/user-attachments/assets/f5c7888a-e206-42aa-bd35-1bf6cfdfb109)

- An IAM role attached to the EC2 instance to handle secure authentication
![image](https://github.com/user-attachments/assets/8bf569e1-63ad-410a-888d-04ef0c299dac)

Once configured, Maven successfully retrieved packages from CodeArtifact using a `settings.xml` file tailored for authentication and repository access.

![image](https://github.com/user-attachments/assets/2ec5b0f0-6a3f-41c9-b941-00aaa57cbc35)

#### Verifying the Connection
After compiling, the CodeArtifact repository was checked, confirming the presence of Maven packages and a successful integration.
![image](https://github.com/user-attachments/assets/1875755e-c0ab-4f9c-abda-18d170812439)

### Continuous Integration with AWS CodeBuild
AWS CodeBuild simplifies and enhances the reliability of builds. In this project, it securely integrates with GitHub via an app, eliminating the need for manual credential management.

![image](https://github.com/user-attachments/assets/5fd99875-97db-41da-8500-d6541932d86c)

The build process is guided by `buildspec.yml`, which defines the setup, tool installation, and code packaging.
![Screenshot 2025-05-07 144842](https://github.com/user-attachments/assets/3ce61412-7a68-43f4-93d8-40e2a90e4872)

Once the build completes, the final files are stored in the S3 bucket `"nextwork-devops-cicd-shanikah"`, with the presence of a WAR file indicating a successful build.

CloudWatch Logs provide real-time visibility into the process, displaying outputs and errors to facilitate quick troubleshooting.  

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

To validate the end-to-end workflow, a front-end change was pushed to GitHub, triggering the pipeline to detect the update, build the application, deploy it, and reflect the changes live, without manual intervention. This automation streamlines the development workflow, reducing deployment time and minimizing human error.

![image](https://github.com/user-attachments/assets/9bae94d5-c77e-42f8-a0ed-cef6bd259d32)


## Key Takeaways
Successfully automated the entire CI/CD workflow using AWS and GitHub

Emphasized security through IAM roles, key-based SSH, and secure token management

Adopted infrastructure as code via CloudFormation for reproducibility

Demonstrated real-time deployment of application changes with zero downtime

## Future Enhancements
Introduce automated testing within the pipeline

Implement blue/green or canary deployments

Add monitoring, alerting, and logging enhancements with CloudWatch and SNS





