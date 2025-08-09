# Automated CI/CD Pipeline with AWS for Java Web Application

This project demonstrates a fully automated CI/CD pipeline using AWS services for a Java-based web application. It replaces slow, error-prone manual deployments with a secure, scalable, and maintainable cloud-native workflow—from source control to production release.

The pipeline integrates four core AWS services: AWS CodePipeline orchestrates the workflow, AWS CodeBuild handles automated builds and tests, AWS CodeDeploy manages zero-downtime deployments, and AWS CodeArtifact secures and centralizes dependency management. GitHub serves as the version control system, triggering the pipeline on code changes. Artifacts are deployed to EC2 instances provisioned via CloudFormation, improving deployment speed, reducing human error, and enhancing traceability.

---

## 🖥Remote Development Environment on EC2

To support remote development, an Amazon EC2 instance was provisioned and secured using an EC2 key pair.

<img width="858" height="169" alt="image" src="https://github.com/user-attachments/assets/07c62618-7062-4b25-9091-95ef8b5d6ffe" />

SSH access was limited to the instance owner, reducing the attack surface and preventing unauthorized access.
 
Development was streamlined using Visual Studio Code’s Remote – SSH extension, allowing direct editing on the EC2 instance without manual file transfers. The environment was configured with Amazon Corretto 8 for Java runtime and Apache Maven for build automation. The project was initialized using Maven’s archetype system, promoting a standardized and modular codebase structure.

<img width="1888" height="631" alt="image" src="https://github.com/user-attachments/assets/cb03eb9d-01d0-4f6f-a747-13d79bb08c2c" />

Version control was established with Git, securely connected to GitHub via a Personal Access Token (PAT). This laid the foundation for seamless integration with the CI/CD pipeline.

---

## 🔐 Secure Dependency Management with AWS CodeArtifact
To establish a secure and consistent foundation for managing Maven packages, AWS CodeArtifact was configured as the central repository. A domain named `nextwork` was provisioned to group related repositories, including `nextwork-devops-cicd`, which used Maven Central as its upstream source. 

<img width="889" height="338" alt="image" src="https://github.com/user-attachments/assets/9b0965ae-6c28-4184-9396-318c65cd54f9" />

This setup enabled access to public packages while enforcing internal controls such as version pinning via `pom.xml`, artifact caching to accelerate builds, and isolation from unverified sources to reduce supply chain risks.

Access to AWS CodeArtifact is managed through an IAM role with read-only permissions for both CodeArtifact and AWS Security Token Service (STS). The role acquires temporary credentials via STS, which are used to generate an authorization token through the AWS CLI. This token is stored in the `CODEARTIFACT_AUTH_TOKEN` environment variable and injected into Maven’s `settings.xml` as a bearer token profile, enabling secure, time-limited access to the repository without relying on static credentials.

Since the token expires after 12 hours, an automated refresh mechanism ensures it is regenerated and re-injected during long-running build processes, maintaining uninterrupted access while minimizing security exposure.

To support long-running build processes, an automated refresh mechanism regenerates and re-injects the token as needed, maintaining uninterrupted access while minimizing credential exposure. With this configuration, Maven retrieves all dependencies directly from CodeArtifact, ensuring that builds are isolated from external sources and aligned with internal governance policies.

<img width="1797" height="970" alt="image" src="https://github.com/user-attachments/assets/546f99e9-441a-4a57-942c-b03c0b30ef6c" />

A post-build verification step confirms the presence and integrity of expected packages, validating compliance with enterprise DevOps standards. This solution delivers a secure, reproducible, and fully automated workflow for dependency management, streamlining builds and strengthening software supply chain security.

---
## ⚙️ Continuous Integration with AWS CodeBuild
To automate the build and deployment process for the Java application, AWS CodeBuild serves as the backbone of the continuous integration (CI) pipeline. This setup compiles, tests, and packages the application into a deployable `WAR` (Web Application Archive) file, all without manual intervention.

The pipeline integrates seamlessly with GitHub via the official GitHub App, triggering builds automatically on every push to the `master` branch. This eliminates static credentials and manual intervention, ensuring a secure and efficient workflow.

Each build runs in a clean, isolated environment defined by the `buildspec.yml` file, progressing through four phases:
- Install: Sets up the Java runtime using Amazon Corretto 8.
- Pre-build: Initializes the environment and retrieves a temporary token to authenticate with AWS CodeArtifact.
- Build: Compiles the application using Maven with a custom `settings.xml`.
- Post-build: Packages the compiled code into a WAR file and prepares deployment assets.

Upon completion, all artifacts, including the `WAR` file, `appspec.yml` and deployment scripts, are bundled into a ZIP archive and securely uploaded to an encrypted Amazon S3 bucket (`nextwork-devops-cicd-shanikah`). 

<img width="1859" height="331" alt="image" src="https://github.com/user-attachments/assets/8a94bc12-84b8-482c-8b60-4c33aad6d3d3" />

Build logs are streamed to Amazon CloudWatch Logs for real-time monitoring and traceability.

<img width="1883" height="945" alt="image" src="https://github.com/user-attachments/assets/c2da06b8-8407-4f43-912f-36a7c5723dc5" />

This CI pipeline enforces consistency, strengthens security and accelerates delivery. By unifying source control, build orchestration, and artifact storage, it enables fast, reliable deployment of production-ready software with minimal overhead.
<img width="1820" height="958" alt="image" src="https://github.com/user-attachments/assets/e52396b3-9121-49d5-bc2f-be748ae339bb" />

---

## 📦 Automated Deployment with AWS CodeDeploy & CloudFormation
To begin the deployment process, a production-ready EC2 instance and its networking infrastructure were launched using AWS CloudFormation. This instance was distinct from the earlier development environment and was dedicated to hosting the live application, ensuring separation between testing and production. CloudFormation, AWS’s Infrastructure as Code (IaC) tool, was used to define and automate the creation of all necessary resources—including a VPC, subnet, route tables, internet gateway, security group, EC2 instance tagged with role=webserver, IAM roles with scoped permissions for CodeDeploy, S3, and Systems Manager, and an encrypted S3 bucket (nextwork-devops-cicd-shanikah) for storing deployment artifacts. The CloudFormation stack was configured with rollback and cleanup options to prevent partial deployments, and the user’s IP address was added with CIDR notation (/32) to restrict SSH access for security. This approach ensured consistent, reproducible infrastructure provisioning across environments, reduced manual errors, and accelerated onboarding. Once submitted, stack creation progress was monitored through CloudFormation’s Events and Resources tabs, providing visibility into each step of the setup.

To complete the CI/CD pipeline, AWS CodeDeploy was configured to orchestrate secure and controlled application deployment to EC2 instances. After successful builds in AWS CodeBuild, application files and lifecycle scripts were stored in the encrypted S3 bucket and retrieved during deployment. Deployment orchestration was defined in the appspec.yml file, which mapped application files and declared lifecycle event hooks. These hooks were supported by custom shell scripts: install_dependencies.sh installed Apache HTTP Server and Apache Tomcat, configuring Apache as a reverse proxy; start_server.sh launched services and enabled auto-restart on reboot; and stop_server.sh gracefully halted services before deployment to ensure clean updates. CodeDeploy deployed assets to EC2 instances provisioned via CloudFormation, with the CodeDeploy agent installed and centrally managed using AWS Systems Manager to enable automated health checks and reduce operational overhead. Deployment success was validated by accessing the EC2 instance’s public endpoint, confirming that the application was correctly installed and running. CodeDeploy’s lifecycle hooks and rollback capabilities enabled safe, phased rollouts with minimal downtime and rapid recovery, delivering automated, repeatable deployments with zero manual steps, secure delivery through IAM roles and encrypted artifact access, and operational resilience through centralized agent management.


---
Deployment automation was achieved using AWS CodeDeploy. After successful builds, application files and lifecycle scripts were stored in S3 and retrieved during deployment. Deployment behavior was defined in the `appspec.yml` file, which mapped files and specified lifecycle hooks for install, start, and stop phases.

These phases were supported by custom shell scripts: `install_dependencies.sh` installed Apache and Tomcat, configuring Apache as a reverse proxy; `start_server.sh` launched services and enabled auto-restart on reboot; and `stop_server.sh` safely halted services before deployment to prevent conflicts.

CodeDeploy was configured with a deployment group targeting EC2 instances tagged with `role=webserver`. IAM roles were assigned for secure access, and the CodeDeploy agent was installed and managed via AWS Systems Manager. Deployment success was verified via the EC2 instance’s public DNS, confirming that all components were correctly installed and functioning.

This setup enabled scalable, tag-based deployments with rollback support and minimal operational overhead.

---

## 🔄 End-to-End Automation with AWS CodePipeline

AWS CodePipeline ties the entire CI/CD process together, integrating GitHub, CodeBuild, and CodeDeploy into a seamless workflow. The pipeline continuously monitors the `master` branch via webhook triggers, enabling near real-time execution whenever new code is pushed.

Upon detecting a change, the pipeline begins with the Source stage, retrieving the latest code from GitHub. It then moves to the Build stage, where CodeBuild compiles and packages the application. Operating in Superseded mode, the pipeline processes only the most recent revision, avoiding redundant builds and conserving resources. IAM roles are automatically provisioned to enforce least-privilege access across services.

In the final stage, CodeDeploy deploys the application to EC2 instances with zero downtime, ensuring uninterrupted access for users. If a deployment fails, CodeDeploy rolls back to the last stable version, preserving production integrity.

A minor code change was used to validate the pipeline’s responsiveness. The successful execution confirmed the reliability and efficiency of the CI/CD strategy.

---

## ✅ Outcome

By automating software delivery with AWS-native tools, this project achieved faster deployment cycles, reduced manual intervention, improved security and traceability, and scalable infrastructure-as-code provisioning. The result is a robust foundation for modern DevOps workflows in cloud-native Java applications.

---

Let me know if you'd like help formatting this into a `README.md` file with badges, licensing, or contribution guidelines—or if you want to break it into modular documentation for GitHub Pages.
































/////


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

### Pipeline Automation with AWS CodePipeline
AWS CodePipeline was configured to automate the entire CI/CD process by integrating GitHub, AWS CodeBuild, and AWS CodeDeploy into a cohesive workflow. The pipeline continuously monitors the master branch in GitHub using webhook triggers, enabling near real-time execution whenever new code is pushed. This ensures that updates are automatically built, tested, and deployed with minimal manual intervention.

Upon detecting a change, the pipeline begins with the Source stage, retrieving the latest code from GitHub. It then moves to the Build stage, where AWS CodeBuild compiles and packages the application. Operating in Superseded mode, the pipeline processes only the most recent revision, avoiding redundant builds and conserving resources. IAM roles are automatically provisioned to grant secure, scoped permissions to each service, reinforcing both automation and security.

In the final stage, AWS CodeDeploy deploys the application to EC2 instances with zero downtime, ensuring uninterrupted access for users. If a deployment fails, CodeDeploy automatically initiates a rollback to the last known stable version, preserving production integrity.

A minor code change was used to validate the pipeline’s responsiveness and correctness. The successful execution confirmed the reliability of the setup and the effectiveness of the CI/CD strategy. By automating software delivery and enforcing best practices, AWS CodePipeline improves development efficiency, reduces operational overhead, and enhances security across the application lifecycle.

![image](https://github.com/user-attachments/assets/9bae94d5-c77e-42f8-a0ed-cef6bd259d32)

 ![image](https://github.com/user-attachments/assets/66dbea8f-0da0-4d12-b4ff-38e627c97e45)


![image](https://github.com/user-attachments/assets/a29c887e-4e74-4052-ac88-c6c0a851c3c6)


![image](https://github.com/user-attachments/assets/b18346a7-4947-4f42-b069-750bf73f83de)


## Lessons Learned and Challenges Overcome
**

## Final Deliverables and Reflection
This project implemented a fully automated CI/CD pipeline for web application deployment on EC2. By integrating AWS CodePipeline, CodeBuild, CodeDeploy, and CodeArtifact, it ensured seamless updates with zero downtime.

Through this experience, I gained deeper expertise in AWS DevOps tools, transforming manual workflows into automated pipelines that improved deployment speed and software stability. The final implementation ensured that modifying the index.jsp file in GitHub automatically triggered the pipeline, instantly delivering updates while maintaining consistent application performance.

![Screenshot 2025-05-07 164933](https://github.com/user-attachments/assets/4e4b85f0-5e4a-4c26-a744-160e209002c8)

## lesson

An early deployment failure caused by outdated scripts highlighted the importance of maintaining alignment between build and deployment artifacts. This experience reinforced the need for thorough, end-to-end validation across the pipeline to ensure consistent, production-ready deployments and reduce the likelihood of runtime errors.

