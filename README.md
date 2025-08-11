# Automated CI/CD Pipeline with AWS for Java Web Application
In modern software development, rapid and reliable delivery is essential. This project was designed to implement a fully automated CI/CD pipeline using AWS services for a Java-based web application. The goal was to eliminate manual deployment bottlenecks, enforce security best practices, and ensure consistent, reproducible builds across environments.

The pipeline integrates several core AWS services. 
- AWS CodePipeline orchestrates the end-to-end CI/CD workflow.
- AWS CodeBuild automates compilation and testing of the application.
- AWS CodeDeploy performs zero-downtime deployments to EC2 instances.
- AWS CodeArtifact secures and centralizes Maven dependency management.
- GitHub acts as the version control system and triggers the pipeline on code changes.
- AWS CloudFormation provisions EC2 infrastructure, improving deployment speed, reducing human error, and enhancing traceability.

---

## 🖥Remote Development Environment on EC2
To support remote development, an Amazon EC2 instance was provisioned and secured using an EC2 key pair.

<img width="858" height="169" alt="image" src="https://github.com/user-attachments/assets/07c62618-7062-4b25-9091-95ef8b5d6ffe" />

SSH access was limited to the instance owner, minimizing the attack surface and preventing unauthorized access.
 
Development was streamlined using Visual Studio Code’s Remote – SSH extension, which enabled direct editing on the EC2 instance without manual file transfers. The environment was configured with Amazon Corretto 8 for the Java runtime and Apache Maven for build automation. The project was initialized using Maven’s archetype system, promoting a standardized and modular codebase structure.

<img width="1888" height="631" alt="image" src="https://github.com/user-attachments/assets/cb03eb9d-01d0-4f6f-a747-13d79bb08c2c" />

Version control was established using Git, securely connected to GitHub via a Personal Access Token (PAT). This setup laid the foundation for seamless integration with the CI/CD pipeline.

---

## 🔐 Secure Dependency Management with AWS CodeArtifact
To establish a secure and consistent foundation for managing Maven packages, AWS CodeArtifact was configured as the central repository. A domain named `nextwork` was provisioned to group related repositories, including `nextwork-devops-cicd`, which used Maven Central as its upstream source. 
<p align="center">
  <img width="700" alt="Descriptive Alt Text" src="https://github.com/user-attachments/assets/9b0965ae-6c28-4184-9396-318c65cd54f9" />
</p>
This setup enabled access to public packages while enforcing internal controls such as version pinning via `pom.xml`, artifact caching to accelerate builds, and isolation from unverified sources to reduce supply chain risks.

Access to AWS CodeArtifact was managed through an IAM role with read-only permissions for both CodeArtifact and AWS Security Token Service (STS). The role acquired temporary credentials via STS, which are used to generate an authorization token through the AWS CLI. This token was stored in the `CODEARTIFACT_AUTH_TOKEN` environment variable and injected into Maven’s `settings.xml` as a bearer token profile. This enabled secure, time-limited access to the repository without relying on static credentials.

Since the token expires after twelve hours, an automated refresh mechanism was implemented to regenerate and re-inject the token during long-running build processes. This ensured uninterrupted access while minimizing credential exposure. With this configuration, Maven retrieved all dependencies directly from CodeArtifact, aligning builds with internal governance policies. 
<p align="center">
  <img width="800" alt="Descriptive Alt Text" src="https://github.com/user-attachments/assets/546f99e9-441a-4a57-942c-b03c0b30ef6c" />
</p>
A post-build verification step confirmed the presence and integrity of expected packages, validating compliance with enterprise DevOps standards.

---
## ⚙️ Continuous Integration with AWS CodeBuild
To automate the build and deployment process for the Java application, AWS CodeBuild served as the backbone of the continuous integration (CI) pipeline. It compiled, tested, and packaged the application into a deployable WAR (Web Application Archive) file, all without manual intervention.

The pipeline integrated seamlessly with GitHub via the official GitHub App, triggering builds automatically on every push to the `master` branch. This eliminated the need for static credentials and manual triggers, ensuring a secure and efficient workflow.

Each build ran in a clean, isolated environment defined by the `buildspec.yml` file. The build process progressed through four phases:
- Install: Sets up the Java runtime using Amazon Corretto 8.
- Pre-build: Initializes the environment and retrieves a temporary token to authenticate with AWS CodeArtifact.
- Build: Compiles the application using Maven with a custom `settings.xml`.
- Post-build: Packages the compiled code into a WAR file and prepares deployment assets.

<img width="1820" height="958" alt="image" src="https://github.com/user-attachments/assets/e52396b3-9121-49d5-bc2f-be748ae339bb" />

Upon completion, all artifacts including the `WAR` file, `appspec.yml` and deployment scripts were bundled into a ZIP archive and securely uploaded to an encrypted Amazon S3 bucket named `nextwork-devops-cicd-shanikah`. 

<img width="1859" height="331" alt="image" src="https://github.com/user-attachments/assets/8a94bc12-84b8-482c-8b60-4c33aad6d3d3" />

Build logs were streamed to Amazon CloudWatch Logs for real-time monitoring and traceability.

<img width="1883" height="945" alt="image" src="https://github.com/user-attachments/assets/c2da06b8-8407-4f43-912f-36a7c5723dc5" />

---
## Provisioning Production Resources Using AWS CloudFormation
The production environment was provisioned using AWS CloudFormation, which deployed a production-grade EC2 instance along with its complete networking infrastructure. This included a virtual private cloud, subnets, route tables, an internet gateway and a security group. The EC2 instance, tagged as webserver, was designed specifically to host the live application and was intentionally separated from the development environment to maintain a clear boundary.

Using infrastructure as code allowed for automated and version-controlled deployments. Rollback and cleanup policies were configured to prevent failed or partial launches, ensuring consistent environments and reducing manual errors.

---

## Application Deployment with AWS CodeDeploy
Once the application was built and packaged by AWS CodeBuild, deployment to the production EC2 instance was automated using AWS CodeDeploy. This completed the continuous integration and delivery pipeline. The EC2 instance, hosted in a dedicated production VPC, ensured isolation from development resources.

The deployment process was triggered directly by CodeBuild as part of the post-build phase. After the `WAR` file, `appspec.yml` and supporting scripts were uploaded to an encrypted Amazon S3 bucket, CodeDeploy initiated the deployment using the AllAtOnce strategy. This approach replaced the existing application in a single step, resulting in brief but controlled downtime. Its simplicity and speed made it well-suited for the lightweight production environment.

Deployment behavior was defined in the `appspec.yml` file, which specified file mappings and lifecycle event hooks. These hooks invoked custom scripts to manage application services:
- `install_dependencies.sh` installed and configured Apache HTTP Server and Apache Tomcat.
- `stop_server.sh` gracefully stopped services before deployment.
- `start_server.sh` restarted services and enabled auto-restart on reboot.

To maintain security, CodeDeploy operated under the `NextWorkCodeDeployRole` IAM role, which provided tightly scoped permissions to access required AWS resources and interact with the target EC2 instance. This ensured secure, auditable, and controlled deployments.

Once deployment was complete, the application was validated by accessing the EC2 instance’s public endpoint, confirming that services were up and responding as expected. 
<p align="center">
  <img width="750" alt="Descriptive Alt Text" src="https://github.com/user-attachments/assets/b7a967af-be52-4ee4-894f-d83d3f4e34df" />
</p>

lll
<img width="942" height="345" alt="image" src="https://github.com/user-attachments/assets/b7a967af-be52-4ee4-894f-d83d3f4e34df" />

---

## 🔄 End-to-End Automation with AWS CodePipeline
AWS CodePipeline orchestrated the entire CI/CD workflow, integrating source control, build automation, dependency resolution, artifact storage, and deployment.

<img width="940" height="443" alt="image" src="https://github.com/user-attachments/assets/83b206fd-0827-4b8e-a72b-b2b76f65c357" />

The pipeline, named `nextwork-devops-cicd`, was built using Pipeline Type V2 and ran in superseded execution mode. This ensured that only the latest code changes were deployed by automatically canceling outdated runs, reducing deployment risk and maintaining consistency across environments.

CodePipeline was triggered by webhook events on the GitHub `master` branch. It delegated the build phase to AWS CodeBuild, which compiled and tested the application while resolving dependencies via AWS CodeArtifact. Build artifacts were stored in Amazon S3, and deployment was handled by AWS CodeDeploy, which updated the EC2 instance.

Each stage was modular and purpose-built. CodePipeline managed execution flow, enforced success criteria and passed artifacts between services. On deployment failure, automatic rollback was triggered via CodeDeploy to maintain system stability. Access control was enforced through the IAM role `AWSCodePipelineServiceRole-us-east-1-nextwork-devops-cicd`, which followed least-privilege principles and provided scoped permissions to all integrated services.

<p align="center">
  <img width="200" alt="image1" src="https://github.com/user-attachments/assets/8f9791ed-b7c6-470b-bc56-23f14e5c508b" />
  <img width="200" alt="image2" src="https://github.com/user-attachments/assets/e52b14ed-798c-4d76-8089-aacc31eb8fd9" />
  <img width="200" alt="image3" src="https://github.com/user-attachments/assets/b121f24c-c1d2-4359-b4fe-c335431fea0d" />
</p>


---
## Overcoming Challenges and Achieving Results
This project delivered a fully automated CI/CD pipeline that eliminated manual interventions, accelerated deployment speed, and boosted reliability. 

<img width="1570" height="414" alt="image" src="https://github.com/user-attachments/assets/4cb046dd-7c12-4928-b15c-74135e8711a9" />

By leveraging IAM roles with temporary credentials, the pipeline ensured robust security while maintaining reproducible and traceable builds across environments.

Through this experience, I advanced my expertise in cloud-native DevOps practices, secure credential management, and infrastructure as code. The result is a production-ready pipeline that enables rapid development cycles, consistent environment provisioning, and secure delivery tailored for Java web applications.

Looking ahead, the pipeline can be further enhanced by implementing blue/green deployment strategies using AWS CodeDeploy. This approach will enable zero-downtime releases and safer rollbacks by seamlessly shifting traffic between application versions. Additionally, expanding to multi-region deployments will improve application availability, reduce latency, and strengthen disaster recovery by distributing infrastructure and workloads across multiple AWS regions. These future enhancements will significantly increase system resilience, scalability, and user experience, positioning the pipeline to support complex, global applications with ease.

---





























