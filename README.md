# Build Infrastructure with Terraform on Google Cloud

The project demonstrates how to provision, import, modify, and manage Google Cloud infrastructure using **Terraform**, including modular configurations, remote state storage, and reusable modules from the Terraform Registry.

---

## Project Overview

In this project, Terraform is used to:

- Import existing Compute Engine instances into Terraform state
- Build reusable Terraform modules
- Configure a remote backend using Cloud Storage
- Provision Google Cloud infrastructure
- Create and manage Virtual Private Cloud (VPC) networks
- Configure firewall rules
- Update and destroy infrastructure
- Use official Terraform Registry modules

---

## Technologies Used

- Terraform
- Google Cloud Platform (GCP)
- Google Compute Engine
- Google Cloud Storage
- Google VPC Networking
- Terraform Registry Modules
- Cloud Shell

---

## Repository Structure

```text
.
├── main.tf
├── variables.tf
├── modules
│   ├── instances
│   │   ├── instances.tf
│   │   ├── outputs.tf
│   │   └── variables.tf
│   └── storage
│       ├── storage.tf
│       ├── outputs.tf
│       └── variables.tf
└── README.md
```

---

# Objectives

## Task 1 — Create Terraform Configuration

Create the project structure and Terraform configuration files.

### Files Created

- `main.tf`
- `variables.tf`
- `modules/instances`
- `modules/storage`

Configure:

- Google Provider
- Project variables
- Region
- Zone

Initialize Terraform:

```bash
terraform init
```

---

## Task 2 — Import Existing Infrastructure

Import two existing Compute Engine instances into Terraform state.

Resources imported:

- tf-instance-1
- tf-instance-2

Commands:

```bash
terraform import module.instances.google_compute_instance.tf-instance-1 tf-instance-1

terraform import module.instances.google_compute_instance.tf-instance-2 tf-instance-2
```

Validate:

```bash
terraform plan

terraform apply
```

---

## Task 3 — Configure Remote Backend

Create a Cloud Storage bucket to store Terraform state remotely.

Terraform backend:

```hcl
backend "gcs" {
  bucket = "YOUR_BUCKET_NAME"
  prefix = "terraform/state"
}
```

Reinitialize:

```bash
terraform init
```

---

## Task 4 — Modify Infrastructure

Update existing VM instances:

- Change machine type to:

```
e2-standard-2
```

Create an additional Compute Engine instance.

Apply changes:

```bash
terraform plan

terraform apply
```

---

## Task 5 — Destroy Infrastructure

Remove the third VM instance by deleting its Terraform configuration.

Apply:

```bash
terraform apply
```

Terraform automatically destroys the removed resource.

---

## Task 6 — Create VPC using Terraform Registry Module

Use the official Terraform Google Network module.

Features:

- Global routing
- Custom VPC
- Two subnets
- Reusable Terraform Registry module

Subnets:

| Name | CIDR |
|-------|------|
| subnet-01 | 10.10.10.0/24 |
| subnet-02 | 10.10.20.0/24 |

---

## Task 7 — Configure Firewall

Create a firewall rule allowing HTTP traffic.

Configuration:

- TCP Port 80
- Source Range

```
0.0.0.0/0
```

---

# Terraform Workflow

Initialize

```bash
terraform init
```

Validate

```bash
terraform validate
```

Preview Changes

```bash
terraform plan
```

Deploy Infrastructure

```bash
terraform apply
```

Destroy Infrastructure

```bash
terraform destroy
```

---

# Google Cloud Resources Created

- Compute Engine VM Instances
- Cloud Storage Bucket
- Virtual Private Cloud (VPC)
- Two Subnets
- Firewall Rule
- Remote Terraform Backend

---

# Skills Demonstrated

- Infrastructure as Code (IaC)
- Terraform Modules
- Terraform State Management
- Remote Backend Configuration
- Import Existing Infrastructure
- Google Cloud Networking
- Compute Engine Management
- Terraform Registry Modules
- Terraform Best Practices

---

# Prerequisites

- Google Cloud Project
- Cloud Shell
- Terraform
- Google Cloud SDK
- Appropriate IAM Permissions

---

# Learning Outcomes

By completing this project, you will gain experience with:

- Building infrastructure using Terraform
- Managing infrastructure state
- Importing existing resources
- Creating reusable Terraform modules
- Using remote backends
- Deploying VPC networks
- Managing Compute Engine instances
- Configuring Google Cloud networking
- Creating firewall rules
- Applying Infrastructure as Code best practices

---

# References

- Terraform Documentation: https://developer.hashicorp.com/terraform/docs
- Google Cloud Terraform Provider: https://registry.terraform.io/providers/hashicorp/google/latest/docs
- Terraform Google Modules: https://github.com/terraform-google-modules
- Google Cloud Documentation: https://cloud.google.com/docs

---

## Author

**Segun Ofe**

Cloud Architect | DevOps Engineer | Terraform Associate | Google Cloud Professional Cloud Architect | AWS Solutions Architect Associate

GitHub: https://github.com/segunofe

LinkedIn: https://linkedin.com/in/segunofe
