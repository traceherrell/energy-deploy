# **Modernizing Application Development at Dominion Energy: Your Guide to OpenShift and DevOps**

Welcome\! We are launching a new initiative to modernize how we build, deploy, and manage applications at Dominion Energy. This guide explains the vision behind this project, introduces the new technologies, and outlines the steps your team can take to get started. Our goal is to empower your team to deliver value faster and more reliably. 🚀

---

## **Project Goals**

Our primary objective is to evolve our development and operations practices to meet modern standards. This initiative is built on three core pillars:

* **Accelerate Delivery and Improve Reliability:** By automating the build and deployment process, we can release new features and bug fixes more quickly and with fewer errors. Containerization with OpenShift provides consistent environments from development to production, eliminating "it worked on my machine" problems.  
* **Standardize on a Modern, Enterprise-Ready Toolchain:** We are adopting a suite of powerful, industry-leading tools. This common platform will break down silos, improve collaboration, and ensure all teams are building on a secure and supported foundation.  
* **Empower Development Teams:** This new workflow gives your team more control and visibility over the entire application lifecycle. With self-service capabilities and automated pipelines, you can focus more on writing code and less on manual deployment coordination.

---

## **The New Workflow: From Code to Cluster**

Our new process is centered around **GitOps**, where your Git repository is the single source of truth for both your application code and its desired state in the cluster. Here’s a high-level overview of the end-to-end flow:

1. **Code Push:** A developer pushes code changes for a .NET application to its repository in **Azure DevOps (ADO) Git**.  
2. **CI Pipeline Triggers:** This push automatically triggers a build pipeline in **ADO Pipelines**.  
3. **Build & Push Image:** The pipeline uses the application's Dockerfile to build a container image. It then pushes this new versioned image to our on-premises **Quay** container registry.  
4. **Update Deployment Repo:** The final step of the pipeline is to automatically update a file in a *separate* deployment repository. This change typically involves updating the image tag to point to the new version that was just pushed to Quay.  
5. **ArgoCD Syncs:** **ArgoCD**, our automated deployment tool running in OpenShift, constantly monitors the deployment repository. It detects the change to the image tag.  
6. **Deploy to OpenShift:** ArgoCD automatically pulls the correct image from Quay and deploys it to the **OpenShift** cluster, ensuring the running application matches the state defined in your deployment repository.

---

## **Key Technologies Explained**

* **OpenShift:** This is our on-premises application platform, built on Kubernetes. Think of it as an operating system for our data center that manages your containerized applications, handling everything from networking to scaling and health monitoring. 📦  
* **Azure DevOps (ADO):** This is our new, centralized platform for hosting Git repositories and defining our automated CI/CD pipelines.  
* **Quay:** Our private, on-premises container registry. It securely stores and distributes the container images your ADO pipeline builds. Think of it as a private Docker Hub for Dominion Energy.  
* **ArgoCD & GitOps:** ArgoCD is the tool that implements the GitOps workflow. It ensures that the live state of your application in OpenShift is always synchronized with the configuration defined in your deployment Git repository.  
* **Dockerfile:** A simple text file that contains the step-by-step instructions for building your application into a standardized container image. Your team will own the Dockerfile for your application.

---

## **Onboarding Your Application: Step-by-Step Guide**

The DevOps team is here to partner with you throughout this process. Here are the steps to get your application running on OpenShift.

### **Step 1: Initial Consultation & Planning**

Contact the DevOps team to schedule an initial meeting. We will discuss your application's architecture, its dependencies, and create a tailored onboarding plan.

### **Step 2: Migrate Source Code to ADO Git**

We will assist your team in migrating your application's source code from TFS to a new Git repository in Azure DevOps. This is the foundation for all future automation.

### **Step 3: Containerize Your Application (Dockerfile)**

Your team will create a Dockerfile for your application. The DevOps team will provide standardized .NET base images and a template Dockerfile to get you started. This file will live alongside your code in the application repository.

### **Step 4: Create Deployment Manifests**

You'll define your application's desired state in OpenShift using Kubernetes manifest files (YAML). These files specify things like which container image to use, how many replicas to run, and what network ports are needed. These will be stored in a **new, separate deployment repository**, which we will help you set up.

### **Step 5: Configure the CI/CD Pipeline**

The DevOps team will work with you to create a standard ADO pipeline for your application. This pipeline will perform the three key tasks: build the image, push it to Quay, and update your deployment repository.

### **Step 6: Deploy with ArgoCD**

Once the pieces are in place, the DevOps team will configure an ArgoCD application that points to your deployment repository. From this point on, ArgoCD will automatically manage all deployments for you.

---

## **Roles & Responsibilities**

Clear roles help us work together effectively.

#### **Your Application Team Owns:**

* Application source code.  
* The Dockerfile used to build your application container.  
* The Kubernetes deployment manifests (e.g., Kustomize or Helm files) that define your application's resources.  
* Application-level configuration and secrets.  
* Troubleshooting application-specific bugs and performance issues.

#### **The DevOps Team Owns:**

* The OpenShift, ADO, Quay, and ArgoCD platforms.  
* Providing standardized pipeline templates and base container images.  
* Assisting with the initial onboarding, migration, and pipeline setup.  
* Managing the underlying cluster infrastructure and security.  
* Platform-level troubleshooting.

