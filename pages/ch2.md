# **Chapter 2** - Provisioning EC2 using CloudShell & Terraform

## 2.1 Overview

In this chapter, you’ll use the Arculus Ground Control Portal to begin adding drone devices (EC2 instances) into your Arculus cluster. Since the Arculus Portal is already deployed from the previous module, our focus here is on onboarding new nodes, approving join requests, and preparing each device to become a mission-capable drone.

You’ll start by navigating to the Add Nodes page in the Arculus UI, where you will generate a custom enrollment script for each drone. This script will be copied into your EC2 instances and executed to initiate a secure cluster join request. Once the request appears in the Device Management Dashboard, you’ll approve the node and verify that it successfully registers as part of the cluster.

By the end of this chapter, you will have all required drone nodes—such as a Surveillance Drone, Supply Drone, Relay Drone, and Mission Controller.

## 2.2 Adding Drone Nodes to the Arculus Cluster

* Log in to the Arculus Ground Control Portal using your instructor-provided credentials.
* From the left sidebar, click Add Nodes.

<p align="center"> <img src="../img/ch2_add_node_left.png" width="900px"></p>


* You’ll see a simple form where you can enter the name of the new device you want to add to the cluster.


<p align="center"> <img src="../img/ch2_addnode_page.png" width="900px"></p>


* In the top search bar, type "CloudShell" and select **CloudShell** from the services list.

<p align="center"> <img src="../img/ch2_CloudShell_search.png" width="900px"></p>

* You will arrive in this CloudShell terminal:

<p align="center"> <img src="../img/Ch2_CloudShell_StartPage.png" width="900px"></p>

## 2.2 Setting up Terraform

Install Terraform into /tmp for this session, reset env vars so it uses CloudShell’s role credentials (no profiles), and point Terraform’s state and plugin cache to /tmp to avoid home-directory quotas—then move into the working directory.

### 2.2.1 Set the AWS region for this session

* This sets ```AWS_REGION``` (defaults to ```us-east-1``` if unset) so all CLI and Terraform calls target the same region.

```bash
export AWS_REGION=${AWS_REGION:-us-east-1}
```
### 2.2.2 Clear any local/profile credentials

* Wipes env vars for profiles/keys/tokens so CloudShell uses its temporary role credentials (clean, least-privilege, no stale keys).
  
```bash
unset AWS_PROFILE AWS_SDK_LOAD_CONFIG AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN
```
### 2.2.3 Install Terraform into /tmp and put it on ``PATH``

* This script figures out what kind of CPU your CloudShell session is using, downloads the matching Terraform program, and unzips it into /tmp/bin. It also creates a working folder for this lab and updates your PATH so that when you type terraform, it uses this freshly downloaded version.
  
```bash
TF_VERSION="1.9.5"
ARCH=$(uname -m); case "$ARCH" in x86_64) TF_ARCH="amd64" ;; aarch64) TF_ARCH="arm64" ;; *) echo "Unsupported arch: $ARCH"; exit 1 ;; esac
mkdir -p /tmp/bin /tmp/arculus/ch3
curl -fsSLo /tmp/terraform.zip "https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_linux_${TF_ARCH}.zip"
unzip -o /tmp/terraform.zip -d /tmp/bin >/dev/null
export PATH="/tmp/bin:$PATH"
```
### 2.2.4 Verify Terraform is available

* Quick health check that prints the Terraform version—confirms the install and ```PATH``` are correct.

* Points Terraform’s working data (``TF_DATA_DIR``) and provider plugin cache (``TF_PLUGIN_CACHE_DIR``) to ``/tmp`` to avoid home-dir quotas and speed up repeated runs.

```bash
terraform -version
```
### 2.2.5 Keep Terraform state/cache in ```/tmp```

* Points Terraform’s working data (```TF_DATA_DIR```) and provider plugin cache (```TF_PLUGIN_CACHE_DIR```) to ```/tmp``` to avoid home-dir quotas and speed up repeated runs.

```bash
export TF_DATA_DIR=/tmp/.tfdata
export TF_PLUGIN_CACHE_DIR=/tmp/.tfplugins
```
### 2.2.6 Prepare cache directory and enter the lab folder

* Ensures the plugin-cache directory exists, then ```cd```s into the working directory where you’ll place ```main.tf``` and run ```init/plan/apply```.

```bash
mkdir -p "$TF_PLUGIN_CACHE_DIR"

# Work directory
cd /tmp/arculus/ch3
```

## 2.3 Main Commands

```bash
terraform init -reconfigure
```

* **What it does:** Creates/refreshes the working directory for Terraform. It:
    * Makes a ```.terraform/``` folder,
    * Downloads provider plugins (e.g., ```hashicorp/aws```) and caches them,
    * (If you use a backend) initializes or re-points the state backend.
* **Why** ```-reconfigure```: Forces Terraform to ignore any saved backend settings and re-read backend config from your files. Use this if you changed the backend or moved directories.
* **Success cues**: “Terraform has been successfully initialized!” and a list of providers/plugins found.

```bash
terraform fmt
```
* **What it does***: Auto-formats all ``.tf`` files to canonical style (spacing, alignment, block ordering).
* **Why it matters**: Prevents formatting drift, merges cleanly, and avoids silly diffs in PRs.
* **Success cues**: It prints the file names it reformatted (or nothing if everything already matched style).


```bash
terraform validate
```
* **What it does:** Static checks your configuration:
    * HCL syntax is valid,
    * Required variables are declared,
    * Resource references line up,
    * Providers required by resources are specified.
* **What it does not do:** It doesn’t contact AWS or evaluate data sources; it’s a local, quick sanity check.
* **Success cues**: “Success! The configuration is valid.”


```bash
terraform apply -auto-approve -var="region=${AWS_REGION}" -var="az=us-east-1a"
```
* **What it does:** Plans and applies changes to reach the desired state (create, update, or destroy resources).
* **Flags explained:**
    * ``-auto-approve`` — Skips the interactive “type yes” approval step. Great for labs/automation; be careful in real environments.
    * ``-var="region=${AWS_REGION}"`` — Injects a value for the input variable region at runtime, using your session’s ``AWS_REGION``.
    * ``-var="az=us-east-1a"`` — Sets the ``az`` variable (often used for subnet/instance placement).
* **What you’ll see:** A resource-by-resource log (“Creating… Still creating… Complete”), then a summary like:
    * ``Apply complete! Resources: 4 added, 0 changed, 3 destroyed.``
    * If you defined ``output`` values, they appear under Outputs—copy these for URLs, IDs, etc.

## 2.4 As a Result

This chapter demonstrated provisioning of an EC2 stack with Terraform from AWS CloudShell—installing Terraform in /tmp, directing Terraform state and plugin cache to /tmp, and relying on CloudShell’s role-based credentials. A minimal, production-style configuration was authored: VPC, public subnet, internet route, a unique egress-only security group, and an Ubuntu 22.04 t2.medium instance. Deployment was initialized, applied, and verified via Terraform outputs and the EC2 console.
