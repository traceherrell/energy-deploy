## **Onboarding Readiness Checklist for Application Teams**

Use this checklist to prepare your application and team for a smooth transition to OpenShift. Answering these questions before our initial consultation will significantly speed up the onboarding process.

---

### **✅ Section 1: Application Analysis**

This section helps us understand the architecture and requirements of your application.

* **\[ \] Application Name:** \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
* **\[ \] Identify Application Dependencies:** List all external services your application needs to function.  
  * *Examples: SQL Server databases, Oracle databases, REST APIs, SOAP services, message queues (MSMQ, RabbitMQ), file shares, etc.*  
  * **Why it's important:** We need to ensure network connectivity is in place from the OpenShift cluster to these dependencies.  
* **\[ \] Map Application Configuration:** Where are your application settings stored?  
  * *Examples: Web.config, App.config, appsettings.json, Windows Registry, etc.*  
  * **Why it's important:** In OpenShift, configuration is externalized into objects called ConfigMaps. We need to know what to extract.  
* **\[ \] Identify All Secrets:** List all sensitive values your application uses. **Do not write down the actual secrets**, just their purpose.  
  * *Examples: Database connection strings, API keys, service account passwords, certificates.*  
  * **Why it's important:** Secrets are managed securely by OpenShift. For now, we'll use the command line to create them, so they must be identified upfront.  
* **\[ \] Analyze Application State:** Is your application stateless or stateful?  
  * **Stateless:** Can run with multiple copies without issues and stores no data on its local disk. (Ideal for containers\!)  
  * **Stateful:** Writes files to the local disk that it needs later, uses in-memory session state that isn't shared, etc.  
  * **Why it's important:** Stateful applications require special storage configurations (Persistent Volumes) that we need to plan for.  
* **\[ \] Document Health Check Endpoints:** Does your application have a URL endpoint that can be used to check its health?  
  * *Example: An endpoint like /api/health that returns an HTTP 200 OK status if the application is healthy.*  
  * **Why it's important:** OpenShift uses health checks to know if your application has started successfully and is running correctly. If one doesn't exist, we'll need to plan on adding one.

---

### **Git Section 2: Source Code & Build Process**

This covers preparing your code for its new home in Azure DevOps.

* **\[ \] Locate Source Code in TFS:** Identify the full server path to your application's source code in Team Foundation Server.  
  * **Why it's important:** This is the starting point for the migration to ADO Git.  
* **\[ \] Document Build Steps:** Write down the steps to build your application from a clean checkout. Are there any special scripts, tools, or manual steps involved?  
  * **Why it's important:** This information is critical for creating the Dockerfile and the automated ADO Pipeline.  
* **\[ \] Clean the Repository:** Check your current repository for any large binary files (.dlls, .exe, test data) that don't need to be in source control.  
  * **Why it's important:** Git is not designed to store large binaries, and cleaning them out now will make the migration and future clones much faster.

---

### **🧑‍💻 Section 3: Team Readiness**

This ensures your team is prepared for the new tools and workflows.

* **\[ \] Designate a Team Point of Contact:** Who will be the primary liaison between your team and the DevOps team during onboarding?  
  * **Name:** \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
* **\[ \] Review Git Fundamentals:** Has your team reviewed the basic concepts of Git?  
  * *We recommend searching for a "Git for TFS Users" guide.*  
  * **Why it's important:** Your team will be using ADO Git for all source control, which is fundamentally different from TFS centralized version control.  
* **\[ \] Understand Container Concepts:** Has your team watched an introduction to Docker and containers?  
  * *A 10-minute video on "What is a container?" is a great start.*  
  * **Why it's important:** Your application will run as a container, and understanding the basics will make development and troubleshooting much easier.

---

### **🚀 Next Steps**

Once you've completed this checklist, you are ready to begin the onboarding process\!

1. **Save or print this completed checklist.**  
2. **Create a ticket in our Jira project** (or email the DevOps Team at \[Insert DevOps Team Email/Contact Info\]).  
3. **Attach the completed checklist** to your request.

We'll review your information and schedule the initial consultation. Welcome aboard\!