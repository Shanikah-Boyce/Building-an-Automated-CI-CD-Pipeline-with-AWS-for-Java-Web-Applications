# Automated CI/CD Pipeline with AWS for Java Web Application
In modern software development, rapid and reliable delivery is essential. This project was designed to implement a fully automated CI/CD pipeline using AWS services for a Java-based web application. The goal was to eliminate manual deployment bottlenecks, enforce security best practices, and ensure consistent, reproducible builds across environments.

The pipeline integrates several core AWS services. AWS CodePipeline orchestrates the overall workflow. AWS CodeBuild handles automated compilation and testing. AWS CodeDeploy manages zero-downtime deployments. AWS CodeArtifact secures and centralizes dependency management. GitHub serves as the version control system and triggers the pipeline automatically on code changes. Artifacts are deployed to EC2 instances provisioned via AWS CloudFormation, which improves deployment speed, reduces human error, and enhances traceability.

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

<img width="889" height="338" alt="image" src="https://github.com/user-attachments/assets/9b0965ae-6c28-4184-9396-318c65cd54f9" />

This setup enabled access to public packages while enforcing internal controls such as version pinning via `pom.xml`, artifact caching to accelerate builds, and isolation from unverified sources to reduce supply chain risks.

Access to AWS CodeArtifact was managed through an IAM role with read-only permissions for both CodeArtifact and AWS Security Token Service (STS). The role acquired temporary credentials via STS, which are used to generate an authorization token through the AWS CLI. This token was stored in the `CODEARTIFACT_AUTH_TOKEN` environment variable and injected into Maven’s `settings.xml` as a bearer token profile. This enabled secure, time-limited access to the repository without relying on static credentials.

Since the token expires after twelve hours, an automated refresh mechanism was implemented to regenerate and re-inject the token during long-running build processes. This ensured uninterrupted access while minimizing credential exposure. With this configuration, Maven retrieved all dependencies directly from CodeArtifact, aligning builds with internal governance policies. 

<img width="1797" height="970" alt="image" src="https://github.com/user-attachments/assets/546f99e9-441a-4a57-942c-b03c0b30ef6c" />

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
# Production Environment
## Provisioning Production Resources Using AWS CloudFormation
The deployment process began with AWS CloudFormation provisioning a production-grade EC2 instance along with its complete networking infrastructure. This included a virtual private cloud, subnets, route tables, an internet gateway, and a security group. The EC2 instance, tagged as “webserver,” was designed specifically to host the live application and was intentionally separated from the development environment to maintain a clear boundary.

Using infrastructure as code, CloudFormation enabled automated and version-controlled deployments. Rollback and cleanup policies were configured to prevent failed or partial launches. This approach ensured consistent environments and reduced manual errors.

---

## Application Deployment with AWS CodeDeploy
After the application was built and packaged by AWS CodeBuild, deployment to the production EC2 instance was automated using AWS CodeDeploy, completing the continuous integration and delivery pipeline. This single EC2 instance, hosted in a dedicated production VPC, ensured isolation from development resources.

The deployment process was triggered directly by CodeBuild as part of the post-build phase. Once the `WAR` file, `appspec.yml`, and supporting scripts were uploaded to an encrypted Amazon S3 bucket, CodeDeploy initiated the deployment using the AllAtOnce strategy. This approach replaced the existing application in a single step, resulting in brief but controlled downtime. Its simplicity and speed made it well-suited for the lightweight production environment.

Deployment behavior was defined in the `appspec.yml` file, which specified file mappings and lifecycle event hooks. These hooks invoked custom scripts to manage application services:
- `install_dependencies.sh` installed and configured Apache HTTP Server and Apache Tomcat.
- `stop_server.sh` gracefully stopped services before deployment.
- `start_server.sh` restarted services and enabled auto-restart on reboot.

To maintain security, CodeDeploy operated under the `NextWorkCodeDeployRole` IAM role, which provided tightly scoped permissions to access required AWS resources and interact with the target EC2 instance. This ensured secure, auditable, and controlled deployments.

Once deployment was complete, the application was validated by accessing the EC2 instance’s public endpoint, confirming that services were up and responding as expected. 

<img width="942" height="345" alt="image" src="https://github.com/user-attachments/assets/b7a967af-be52-4ee4-894f-d83d3f4e34df" />

---

## 🔄 End-to-End Automation with AWS CodePipeline
This project uses AWS CodePipeline to orchestrate a fully automated CI/CD workflow, integrating source control, build automation, dependency resolution, artifact storage, and deployment.

The pipeline`nextwork-devops-cicd`—is built using Pipeline Type V2 and runs in superseded execution mode, ensuring only the latest code changes are deployed by automatically canceling outdated runs. This design reduces deployment risk and maintains consistency across environments.

CodePipeline is triggered by webhook events on the GitHub `master` branch. It then delegates the build phase to AWS CodeBuild, which compiles and tests the application while resolving dependencies via AWS CodeArtifact. Build artifacts are stored in Amazon S3, and deployment is handled by AWS CodeDeploy, which updates the EC2 instance.

Each stage is modular and purpose-built, with CodePipeline managing execution flow, enforcing success criteria, and passing artifacts between services. On deployment failure, automatic rollback is triggered via CodeDeploy to maintain system stability.

Access control is enforced through the IAM role `AWSCodePipelineServiceRole-us-east-1-nextwork-devops-cicd`, which follows least-privilege principles and provides scoped permissions to all integrated services.

---
## Conclusion
This project successfully implements a fully automated CI/CD pipeline using AWS services for a Java-based web application. By integrating AWS CodePipeline, CodeBuild, CodeDeploy, and CodeArtifact, the pipeline ensures rapid, repeatable, and secure software delivery from source control to production. Manual steps were eliminated, reducing human error, improving deployment speed, and enforcing consistent environments through infrastructure as code.

Through the use of IAM roles, temporary credentials, and artifact verification, the solution also emphasizes security and governance—critical for enterprise-grade DevOps workflows. With the pipeline in place, each release is not only faster and more reliable but also aligned with cloud-native best practices.

### Personal reflection
This project deepened my expertise in AWS DevOps services, reinforced the value of automation in reducing operational risk, and taught me the importance of designing for both speed and security from the start. The experience also strengthened my problem-solving skills—especially in diagnosing deployment issues and designing resilient token management solutions.

---
To orchestrate the complete CI/CD workflow, AWS CodePipeline serves as the automation backbone. The pipeline (`nextwork-devops-cicd`) is configured with Pipeline Type V2 and operates in superseded execution mode, which automatically cancels any in-progress runs when a newer change is detected. This mechanism ensures optimal resource utilization and maintains deployment consistency.

A dedicated Amazon S3 bucket functions as the default artifact store, while the pipeline is securely governed by the IAM role `AWSCodePipelineServiceRole-us-east-1-nextwork-devops-cicd`. This role enables seamless and secure integration with AWS CodeBuild, CodeDeploy and Amazon S3, facilitating a robust and scalable deployment process.

<img width="940" height="443" alt="image" src="https://github.com/user-attachments/assets/83b206fd-0827-4b8e-a72b-b2b76f65c357" />

The pipeline is structured into three main stages:

<img width="834" height="503" alt="image" src="https://github.com/user-attachments/assets/811769e5-646b-4752-9994-66a81db97cf2" />

### Source Stage
This stage continuously monitors the master branch on GitHub. When changes are pushed, it automatically triggers the pipeline, generating a default artifact for downstream stages.

<img width="491" height="507" alt="image" src="https://github.com/user-attachments/assets/8f9791ed-b7c6-470b-bc56-23f14e5c508b" />

### Build Stage
CodeBuild compiles the source, runs tests, and packages the application for deployment. Using a managed build project ensures consistency and scalability across builds.

<img width="634" height="847" alt="image" src="https://github.com/user-attachments/assets/e52b14ed-798c-4d76-8089-aacc31eb8fd9" />

## Deploy Stage
AWS CodeDeploy handles deployment to the target environment. Rollback is automatically triggered on deployment failure, providing a safety net for production stability.

<img width="636" height="644" alt="image" src="https://github.com/user-attachments/assets/b121f24c-c1d2-4359-b4fe-c335431fea0d" />

By integrating AWS-native services into a unified CI/CD pipeline, this implementation delivers rapid release cycles, robust security controls, and scalable infrastructure-as-code provisioning. It provides a resilient foundation for modern DevOps workflows, enabling continuous delivery of cloud-native Java applications with minimal manual overhead.

<img width="1570" height="414" alt="image" src="https://github.com/user-attachments/assets/4cb046dd-7c12-4928-b15c-74135e8711a9" />

---





























