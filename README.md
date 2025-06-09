# Automated CI/CD Pipeline with AWS for Java Web Application
![image](https://github.com/user-attachments/assets/5ad24dfe-b2c7-4063-8994-ce73112a309f)
## Project Overview
In this project, the goal was to set up a fully automated CI/CD pipeline using AWS services for a Java-based web app. The main aim was to create a reliable, secure and easy-to-manage workflow, covering everything from source control to deployment with modern cloud tools.

The pipeline was built using four key AWS services:

- AWS CodePipeline for orchestrating the entire process 
- AWS CodeBuild for efficient and automated builds  
- AWS CodeDeploy for streamlined and reliable deployments
- AWS CodeArtifact for secure dependency management 

GitHub was used for version control, ensuring that any changes automatically triggered the pipeline, leading to deployments on an EC2 instance provisioned through CloudFormation. This made software updates faster and more efficient.
 
### Environment Setup
The development process started with setting up a remote EC2 instance, ensuring secure SSH access by generating a key pair for authentication. 
![image](https://github.com/user-attachments/assets/e2106c4e-0e46-46d4-8682-7ece63271389)

With the "Remote – SSH extension" in Visual Studio Code, remote development felt as seamless as working locally.
![image](https://github.com/user-attachments/assets/4ff35c90-5381-4851-8293-dd4ca76d2f61)

To establish the core environment, Java and Maven were installed on the EC2 instance. The application’s framework was then structured using Maven’s archetype feature, which provided a standardized directory layout for both code and web assets.
![image](https://github.com/user-attachments/assets/5fa2346f-f8d4-41c1-abd4-c835ae3db323)



![image](https://github.com/user-attachments/assets/7cc384d7-5d2d-4273-abb2-4ccd46e3f133)

### Source Control Integration
A GitHub repository was created and linked to the EC2 instance. Git was configured for proper author attribution and history tracking. Secure access was ensured using a GitHub Personal Access Token (PAT), enabling seamless push/pull operations from the remote environment.

### Dependency Management with AWS CodeArtifact
To securely manage dependencies, AWS CodeArtifact was configured with:
- A custom domain and repository
- Maven Central as the upstream source
- An IAM role attached to the EC2 instance to handle authentication

Once properly configured, Maven successfully pulled packages from CodeArtifact using a `settings.xml` file.

![image](https://github.com/user-attachments/assets/f5c7888a-e206-42aa-bd35-1bf6cfdfb109)


![image](https://github.com/user-attachments/assets/8bf569e1-63ad-410a-888d-04ef0c299dac)

![image](https://github.com/user-attachments/assets/2ec5b0f0-6a3f-41c9-b941-00aaa57cbc35)


![image](https://github.com/user-attachments/assets/1875755e-c0ab-4f9c-abda-18d170812439)

### Continuous Integration with AWS CodeBuild
AWS CodeBuild was set up to automate the build process:
- Integrated directly with GitHub using a GitHub App connection
- Used a `buildspec.yml` file to define environment setup, dependency resolution, compilation, and packaging
- Output artifacts were uploaded to S3

Initial builds highlighted configuration issues, which were resolved by updating IAM permissions and fixing artifact paths.

![image](https://github.com/user-attachments/assets/5fd99875-97db-41da-8500-d6541932d86c)


![image](https://github.com/user-attachments/assets/a595003a-cbe0-4499-8e4d-f45036f29c2e)

![image](https://github.com/user-attachments/assets/3fb0b249-38ca-44d1-a0e1-faecbfd9caa3)

### Automated Deployment with AWS CodeDeploy
Infrastructure resources (EC2, VPC, networking) were defined using CloudFormation for repeatability. Deployment lifecycle was controlled with:
- Custom scripts (install_dependencies.sh, start_server.sh, stop_server.sh)
- An appspec.yml to coordinate execution

A CodeDeploy deployment group targeted EC2 instances using tags. After deployment, the application was verified through the public DNS.
![image](https://github.com/user-attachments/assets/7c8da34c-ba9d-43c6-8993-29ad65bd2df6)


![image](https://github.com/user-attachments/assets/e30a7819-3c34-45e2-b606-7bd00578627f)


![image](https://github.com/user-attachments/assets/ef2b3d82-4963-4a6d-8e47-4a39a643f435)


![image](https://github.com/user-attachments/assets/071187cc-745b-480a-aa42-27f1df968634)

### Pipeline Automation with AWS CodePipeline
A three-stage pipeline was implemented:
- Source: Monitored GitHub for changes
- Build: Triggered CodeBuild for compilation and packaging
- Deploy: Used CodeDeploy to update the EC2 instance

The pipeline operated in Superseded mode, ensuring only the latest changes were processed. IAM roles were auto-generated to grant necessary permissions.

To validate the end-to-end flow, a front-end change was pushed to GitHub. The pipeline automatically detected the change, built the app, deployed it, and reflected the update live without manual intervention.
![image](https://github.com/user-attachments/assets/9bae94d5-c77e-42f8-a0ed-cef6bd259d32)


![image](https://github.com/user-attachments/assets/66dbea8f-0da0-4d12-b4ff-38e627c97e45)


![image](https://github.com/user-attachments/assets/a29c887e-4e74-4052-ac88-c6c0a851c3c6)


![image](https://github.com/user-attachments/assets/b18346a7-4947-4f42-b069-750bf73f83de)

## Key Takeaways
Successfully automated the entire CI/CD workflow using AWS and GitHub

Emphasized security through IAM roles, key-based SSH, and secure token management

Adopted infrastructure as code via CloudFormation for reproducibility

Demonstrated real-time deployment of application changes with zero downtime

## Future Enhancements
Introduce automated testing within the pipeline

Implement blue/green or canary deployments

Add monitoring, alerting, and logging enhancements with CloudWatch and SNS





