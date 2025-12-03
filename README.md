# Configuring Drone Missions on the Arculus Testbed

This repository contains the complete lab module, documentation, and reference materials for **Configuring Drone Missions** on the **Arculus Zero-Trust Edge Security Testbed**.  
This module is a **direct continuation** of the previous Terraform-based module, where students deployed the Arculus Portal and backend infrastructure on AWS.

In this lab, students will enroll EC2-based drone nodes, configure trusted device roles, plan missions, generate encrypted mission manifests, and execute full mission simulations using the Arculus Mission Execution Dashboard.

---

## 📌 Module Overview

This module teaches students how to use Arculus as a secure “mission control tower” for distributed drone devices. Students will learn how drones authenticate into the Arculus cluster, how Zero-Trust rules are enforced at the device and mission levels, and how Arculus dynamically applies and removes network policies during mission execution.

Students will also explore DDIL (Denied, Degraded, Intermittent, Limited) scenarios such as GPS spoofing, communication drops, DoS attempts, physical hijacking, and more.

---

## 📚 What You Will Learn

By completing this lab, students will be able to:

- Add EC2-based drone nodes to the Arculus cluster  
- Approve join requests and configure trusted drone types  
- Assign capabilities and privileges to each drone  
- Plan drone missions and select map destinations  
- Generate encrypted mission manifest (`.mconf`) files  
- Detect and prevent manifest tampering  
- Execute missions using the Arculus UI  
- Monitor telemetry, logs, and dynamic network policies  
- Simulate adversarial conditions during execution  
- Explain how Arculus implements task-based access control (TBAC)

---

## 🧩 Prerequisites

Before starting this module, students must have:

- The **Arculus Portal deployed** using the previous Terraform module  
- Access to AWS (CloudShell preferred for consistency)  
- 3–4 EC2 instances to be used as drone nodes  
- Basic Linux, SSH, and Git knowledge  
- Beginner understanding of JSON/HCL  
- Basic networking knowledge (CIDR, ports, ingress/egress rules)

---

## 🛠️ Repository Structure

