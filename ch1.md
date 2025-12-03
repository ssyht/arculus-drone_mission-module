<a name = "Pg1"></a>

# **Chapter 1** - Overview

## 1.1 Purpose of the Lab
In this lab, you will learn how to use the Arculus Ground Control Portal to configure, manage, and execute secure drone missions on the Arculus edge-security testbed. Building on the Terraform module you completed earlier, this lab assumes your Arculus Portal and controller node are already provisioned and running.

You will explore how to onboard new drone devices (EC2 instances), securely join them to the Arculus cluster, configure each device with trusted drone roles and capabilities, create mission manifests, and launch fully-simulated missions with live monitoring. You’ll also learn how Arculus enforces Zero-Trust through role-based access control, device attestation, dynamic network policies, and mission-driven privilege enforcement (TBAC).

## 1.2 Prerequisites
To follow along and get the most out of this module, you should:

* Have the Arculus Portal fully deployed using the previous Terraform module.

* Have three or more EC2 instances available to use as drones (or be ready to provision them).

* Understand basic AWS concepts (EC2, key pairs, security groups).

* Be comfortable using a terminal (to run join scripts on each drone EC2).

* Have access to Git and the ability to SSH into EC2 instances.

* Have basic familiarity with the Arculus Portal interface (user management, device management).

* Understand JSON/HCL at a beginner level.

* Have a working knowledge of networking basics (CIDR, ports, inbound/outbound rules).


## 1.3 References to guide lab work
Please use the links below to learn the related information for this lab. 

* <a href = "https://developer.hashicorp.com/terraform/tutorials">*Terraform by HashiCorp*</a> - Official docs and tutorials to learn Infrastructure as Code with Terraform
* <a href = "https://registry.terraform.io/providers/hashicorp/aws/latest">*Terraform AWS Provider*</a> - Registry page documenting every AWS resource you can manage with Terraform, plus usage examples.
* <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html">*AWS CLI*</a> - Command-line tool to configure credentials and run AWS service commands from your terminal.
* <a href ="https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html">*AWS Systems Manager SSM*</a> - Secure, logged shell access to instances via browser/CLI without SSH keys or open ports.
* <a href = "https://docs.aws.amazon.com/kms/latest/developerguide/overview.html">*AWS KMS*</a> - Managed service for creating, managing, and using encryption keys and cryptographic operations.
* <a href = "https://aws.amazon.com/">*AWS (Amazon Web Services)*</a> - A cloud platform that offers a variety of services, including storage, compute, and databases. AWS allows users to build and run applications on-demand.

## 1.4 Overview:

In this lab, we’ll use Terraform to stand up a secure, repeatable AWS scaffold for the Arculus edge-security testbed and learn the core ideas of Infrastructure-as-Code (IaC).

**What is Terraform?** A tool to declare infrastructure (VPCs, subnets, security groups, EC2, IAM, KMS, SSM, Secrets Manager) in code. You write the desired state; Terraform plans the changes and applies them safely.

**What is Arculus?** Arculus is a lightweight edge-security platform and web portal that we’ll use as a teaching testbed for Zero-Trust. You can think of it as a “security control tower” in front of your Linux nodes and services: every machine must check in, prove who it is, and then is only allowed to talk to what the policy permits. Through a simple browser UI you enroll nodes, give them identities, set basic “who can talk to whom” rules, and watch live logs that show which connections are allowed or blocked. In this lab, you’ll use Arculus as a visual way to see network security decisions that are usually hidden, without needing deep security expertise.

**Why Terraform for this module?**

* **Repeatable**: One command recreates the same environment for every student/instructor.

* **Safe changes**: Plans show what will change before it changes.

* **Composable**: Modules let us reuse patterns (network, instance, guardrails).

* **Cloud-native security**: Enforce least-privilege IAM, permission boundaries, SSM-only access, KMS encryption, and tight security groups from day one.

## 1.5 Conceptual Overview

<p align="center"> <img src="../img/ch1_overview.png" width="900px"></p>

<p align="center"> <i> Figure 1.1: This figure illustrates the high-level architecture of Terraform. On the left, a DevOps/Infrastructure engineer writes Terraform manifest files (.tf) and runs commands such as terraform plan and terraform apply. These files are processed by the Terraform Core, which loads plugins for different cloud providers (AWS, Azure, GCP, VMware, OpenShift, etc.) and uses provisioners like remote-exec and local-exec when needed. Terraform Core also maintains a state file (.tfstate) that records the current infrastructure. Using the provider plugins and the state file, Terraform communicates with the cloud service providers on the right to create, update, or destroy the actual resources in a consistent, repeatable way.</a> </i> </p>


<p align="center"> <img src="../img/ch1_arculus_concept.png" width="500px"></p>

<p align="center"> <i> Figure 1.2: This figure provides a conceptual overview of the Arculus zero-trust edge testbed. At the top, the Arculus Frontend (left) is where operators interact with the system: they manage users and devices, configure missions, visualize data, and perform other admin controls through a web UI. These actions flow into the Arculus Backend (right), a set of Flask/Node services running on a lightweight K3s cluster that handle client requests, process data logic, retrieve and manage stored information, and enforce security measures. In the center, Arculus weaves in OpenZiti for continuous zero-trust authentication and Wazuh for continuous monitoring, so every connection and event is checked and logged. The middle layer shows the data services (e.g., MySQL) that store OpenZiti identity data, Wazuh logs, real-time network metrics, and MAVLink telemetry. At the bottom, distributed nodes stream sensor data and network traffic into the platform via APIs (OpenZiti, MAVLink, Wazuh), closing the loop between edge devices, secure networking, and the Arculus control dashboard.</a> </i> </p>


## 1.6 Goals/Outcomes:
By the end of this lab module, you will be able to:

(i)	Understand the Terraform Worflow & State

* Explain the plan/apply/destroy lifecycle and why state is critical.

(ii) Create a Minimal, Secure AWS Baseline with IaC

* Provision a VPC with public/private subnets, NAT, and route tables.
* Set up security groups with tight ingress/egress and tags for policy-as-code.
* Launch EC2 instances that are SSM-managed (no public SSH), with hardened ``user_data`` bootstrapping.

(iii) Apply Guardrails & Least Privilege IAM

* Author IAM roles/policies for EC2/SSM that follow least-privilege.
* Understand and attach permission boundaries for student roles in a teaching environment.
* Restrict administration to SSM Session Manager (no public keys, no open SSH).

(iv) Modularize Infrastructure

* Convert repeatable resources into Terraform modules (e.g., network, compute, logging).
* Parameterize modules with sensible defaults and input validation.

(v) Build & Publish a Minimal Lab UI

* Deploy a simple web UI (static HTML/CSS/JS) on the EC2 instance.
* Expose the service on TCP/3000 with a narrowly scoped security group during testing, then tighten to a /32 allowlist (or disable public ingress entirely).
* Use SSM port-forwarding as a no-public-ingress option to reach the UI securely.
* Verify service health with curl/browser, capture a screenshot, and document the URL/port and access method.

(vi) Prepare for Arculus Testbed Integration

* Expose outputs (private IPs, instance profiles, SSM target tags) that later chapters use to deploy Arculus services.
* Align tagging strategy (e.g., Project=Arculus, Env=Lab, Owner=StudentID) for cost visibility and policy targeting.
