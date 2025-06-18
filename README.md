# AWS Infrastructure Provisioning and Application Deployment using Ansible

This project demonstrates the automation of infrastructure provisioning and application deployment on AWS using Ansible. It was built by following a comprehensive Udemy DevOps course and applying the concepts in a hands-on, customized way. The Ansible playbooks are structured to set up a complete VPC architecture and deploy services on EC2 instances, with secure networking and scalability in mind.

While the core structure and logic of the playbooks were developed independently, certain configuration files such as application.properties.j2, the Nginx template, and some database deployment steps—were adapted from course materials to suit the project needs. This experience significantly deepened my understanding of Infrastructure as Code (IaC), dynamic provisioning, and cloud automation practices.


## Project Overview

The goal of the project was to provision a complete AWS architecture using Ansible, involving:

- Creation of a custom **VPC architecture**
- Deploying various **EC2 instances** for backend services
- Setting up a **Bastion host** to access private instances
- Automating the **entire deployment process** from infrastructure to app deployment
- Hosting the app behind a **Load Balancer** and exposing it to the internet

---
## Project Architecture
<p align="center">
  <img src="https://i.imgur.com/P1ctdYv.png" height="80%" width="80%" alt="Geolocation Lookup"/>
</p>
<p align="center">
  <img src="https://i.imgur.com/hG41BU8.png" height="80%" width="80%" alt="Geolocation Lookup"/>
</p>
<p align="center">
  <img src="https://i.imgur.com/nzV6Z9e.png" height="80%" width="80%" alt="Geolocation Lookup"/>
</p>

</br>

##  Steps and Implementation Details

### 1. **Initial Setup and Ansible Controller**

- An **EC2 instance** was launched using **UserData** to serve as the **Ansible Controller**.
- The instance was assigned an **IAM role with Admin access** to allow it to provision AWS resources using Ansible and Boto3 modules.
- Essential Python libraries like `boto3` and `botocore` were installed to enable AWS interactions.

---

### 2. **Writing the VPC Architecture Playbook**

- I wrote the `vpc-setup.yml` playbook to automate the creation of the full VPC structure.
- The playbook provisions:
  - A custom **VPC**
  - Multiple **public and private subnets**
  - **Internet Gateway** and **Route Tables**
  - **NAT Gateways** for enabling outbound internet access from private subnets
- Used `set_fact` module to **dynamically allocate variables** like subnet IDs and store outputs.

To maintain clean and reusable code, I stored output variables in separate variable files under `vars/output_vars`.

---

### 3. **Bastion Host Configuration**

- A playbook named `bastion-instance.yml` was created to provision a **Bastion Host** in the **public subnet**.
- This host provides SSH access to the instances deployed in private subnets.
- Faced some issues with the `ec2` module (Ansible module error: couldn’t resolve ec2), so I manually provisioned the Bastion Host and adjusted my automation accordingly.

---

### 4. **Provisioning the EC2 Stack**

- Wrote the `vpro-ec2-stack.yml` playbook to launch EC2 instances for different services:
  - **Tomcat (App Server)**
  - **MySQL Database**
  - **Memcache**
  - **RabbitMQ**
  - **Nginx (Web Server)**

- Configured an **Application Load Balancer (ALB)** to route traffic to the Nginx instances.
- Used Ansible’s `block` module to group tasks and handle instance creation, tagging, and provisioning.
- As each EC2 instance launched, its **hostname and private IP** were captured and dynamically written into a custom **inventory file** stored in `provision-stack/inventory-vpro`.

---

### 5. **Recreating the Controller in New VPC**

- Since the controller was originally created outside the newly provisioned VPC, I created an **AMI (Image)** of the controller.
- Then, I launched a new instance from this AMI inside the **correct VPC** for secure access to internal services.
- Updated all related variables, inventory paths, and Jinja2 templates to reflect the new environment.

---

### 6. **Templates and Reusable Configurations**

- Utilized **Jinja2 templates** for configuration files:
  - `application.properties.j2`: Copied from the Udemy course and modified to suit the Tomcat app deployment.
  - `nginxvpro.j2`: Template for Nginx configuration, also adapted from the course.

These templates made the deployments consistent and parameterized.

---

### 7. **Master Playbook for Orchestration**

- Created a **`site.yml`** master playbook to manage the complete execution flow.
- It imports and runs all other playbooks in the correct order:
  1. `build.yml` – VPC & networking
  2. Host mapping to inventory file
  3. `db.yml` – Provision database
  4. `dbdeploy.yml` – Deploy schema and sample data (partially referenced from the course)
  5. `memcache.yml`, `rabbitmq.yml` – Set up caching and messaging layers
  6. `appserver.yml` – Deploy the Tomcat app server
  7. `web.yml` – Set up Nginx and Load Balancer

Each step was tested and debugged using **Git Bash** from the local system.

---

### 8. **Final Testing and DNS Mapping**

- After successful deployment, I retrieved the **Load Balancer DNS** from AWS.
- Updated the **DNS record in GoDaddy** to map my domain to the ALB DNS.
- Verified the full end-to-end deployment: app was live and accessible via the public domain.

---

### 9. **Best Practices Followed**

- Used a `.gitignore` file to avoid pushing sensitive and unnecessary files:
  ```gitignore
  *.pem
  *.sql
  *.war

## Lessons Learned

This project helped me gain hands-on experience with:

- **Infrastructure as Code (IaC)** principles using Ansible  
- Writing and debugging **Ansible playbooks** for real-world AWS resources  
- Handling **dynamic inventories**, **Jinja2 templating**, and **variable management**  
- Understanding and applying **secure network architecture design** with Bastion hosts and private subnets  
- Troubleshooting **module-related issues** (`ec2`, `boto3`) and finding workarounds  

---

##  Credits

This project was developed with the guidance of a **Udemy DevOps course** that provided valuable insights, especially:

- Sample **playbooks for deployment flow**
- Templates like `application.properties.j2` and `nginxvpro.j2`
- **Database deployment tasks** and **schema setup**

Special thanks to the instructor for the course structure and examples, which served as a foundation for this custom implementation.

