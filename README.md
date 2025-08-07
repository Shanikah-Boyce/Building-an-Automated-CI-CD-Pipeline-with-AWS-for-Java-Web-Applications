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
 
### Remote Development Environment on EC2
To establish a secure and efficient development environment, I provisioned an Amazon EC2 instance as a remote development server. SSH access was tightly controlled using an EC2 key pair, ensuring that only authorized users (the owner) could connet.  
![image](https://github.com/user-attachments/assets/e2106c4e-0e46-46d4-8682-7ece63271389)
To streamline development workflows, I leveraged Visual Studio Code’s Remote – SSH extension, enabling seamless in-editor access to the remote environment, eliminating the need for manual file transfers or external SSH clients.

The environment was configured with Amazon Corretto 8 and Apache Maven to meet the project’s Java runtime requirements. I initialized the project using Maven’s archetype system, promoting a standardized and modular codebase structure. 

![image](https://github.com/user-attachments/assets/5fa2346f-f8d4-41c1-abd4-c835ae3db323)

Version control was set up with Git, securely connected to GitHub via a Personal Access Token (PAT), laying the groundwork for smooth integration into a CI/CD pipeline.

### Secure Dependency Management with AWS CodeArtifact
To enforce secure and consistent dependency management, AWS CodeArtifact was configured as the central Maven repository for Java builds. This eliminated environment-specific issues by ensuring all dependencies were resolved from a single, controlled source.

A domain (nextwork) and repository (nextwork-devops-cicd) were created in CodeArtifact, with Maven Central set as the upstream. This enabled access to public packages while applying internal controls such as artifact caching, version enforcement, and isolation from unverified sources.

![image](https://github.com/user-attachments/assets/f5c7888a-e206-42aa-bd35-1bf6cfdfb109)

Authentication was handled via an IAM role attached to the EC2 build instance. Using AWS Security Token Service (STS), the role generated temporary tokens that were injected into Maven’s settings.xml and environment variables at runtime—eliminating static credentials and reducing the attack surface.

![image](https://github.com/user-attachments/assets/2ec5b0f0-6a3f-41c9-b941-00aaa57cbc35)

Once configured, Maven successfully pulled dependencies from CodeArtifact during the build process. Post-build verification confirmed that the expected packages were stored in the repository, demonstrating full integration. 

![image](https://github.com/user-attachments/assets/1875755e-c0ab-4f9c-abda-18d170812439)

This setup provided a secure, automated, and reproducible dependency pipeline aligned with enterprise DevOps standards.

### Continuous Integration with AWS CodeBuild
To optimize the build and deployment process, AWS CodeBuild was integrated with GitHub using the official GitHub app. This setup eliminated manual credential management and enhanced security by avoiding personal access tokens and SSH keys. With GitHub serving as the source provider, builds were automatically triggered on code changes, enabling a smooth and efficient CI/CD pipeline.

![image](https://github.com/user-attachments/assets/5fd99875-97db-41da-8500-d6541932d86c)

The build process was defined through a structured buildspec.yml file, outlining each phase of the lifecycle. 
![Screenshot 2025-05-07 144842](https://github.com/user-attachments/assets/3ce61412-7a68-43f4-93d8-40e2a90e4872)


An AWS-managed Corretto 8 image was selected for the build environment to ensure compatibility with the production runtime and maintain consistency across environments. During execution, secure authentication tokens were retrieved for AWS CodeArtifact, followed by Maven dependency resolution, compilation, and packaging.

Upon successful completion, output artifacts—including a WAR file, were securely stored in the S3 bucket nextwork-devops-cicd-shanikah. These files were zipped and prepared for deployment. CloudWatch Logs were enabled to provide real-time visibility into the build process, allowing for rapid issue detection and resolution.

By leveraging CodeBuild’s native integration with AWS services and customizing the environment, the pipeline was fully automated. The result was a secure, reliable, and efficient CI/CD system that accelerated deployment timelines, reduced manual overhead, and ensured consistent, production-ready builds.

### Automated Build & Deployment Pipeline with AWS CodeBuild, CodeDeploy and CloudFormation
To complete the CI/CD pipeline, deployment automation was implemented using AWS CodeDeploy. After successful builds in CodeBuild, the resulting artifacts—including application files and lifecycle scripts—were stored in Amazon S3. These artifacts were then retrieved during deployment, ensuring a consistent and reliable transition from build to release.

Deployment orchestration was defined in the `appspec.yml` file, which mapped files and specified lifecycle hooks for phased operations such as install, start, and stop. 
![image](https://github.com/user-attachments/assets/e30a7819-3c34-45e2-b606-7bd00578627f)

These phases were supported by custom shell scripts: 
- `install_dependencies.sh` installed Apache and Tomcat, with Apache configured as a reverse proxy to Tomcat
- `start_server.sh` ensured services launched automatically and restarted on reboot
- `stop_server.sh` safely halted services to prevent deployment conflicts

To enable targeted and scalable deployments, an AWS CodeDeploy application and deployment group were configured with appropriate IAM roles. EC2 instances were tagged with `role=webserver`, allowing CodeDeploy to automatically include any new instances with matching tags. The CodeDeploy agent was installed on each EC2 instance and managed via AWS Systems Manager, ensuring long-term reliability and reducing operational overhead.

Deployment success was verified through the EC2 instance’s Public IPv4 DNS, confirming that all components were correctly installed and functioning as expected. 
![Screenshot 2025-05-07 160512](https://github.com/user-attachments/assets/f1817f5b-b9b9-446a-803f-3ca0ce97b683)

CodeDeploy’s tight integration with EC2 and support for lifecycle hooks enabled controlled, phased rollouts that minimized risk and improved deployment confidence.

///

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

## lesson

An early deployment failure caused by outdated scripts highlighted the importance of maintaining alignment between build and deployment artifacts. This experience reinforced the need for thorough, end-to-end validation across the pipeline to ensure consistent, production-ready deployments and reduce the likelihood of runtime errors.

