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

Install Terraform

```bash
cat <<'EOF' > ~/.customize_environment
# Set up HashiCorp repository and install Terraform
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install -y terraform
EOF
bash ~/.customize_environment
```

Verify

```bash
terraform --version
```

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

```bash
touch main.tf variables.tf
mkdir -p modules/instances modules/storage


touch modules/instances/instances.tf modules/instances/outputs.tf modules/instances/variables.tf
touch modules/storage/storage.tf modules/storage/outputs.tf modules/storage/variables.tf
```

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

#### 1. Update these variables with your lab values

```bash

REGION="us-east4"       # Replace with your lab region
ZONE="us-east4-c"       # Replace with your lab zone
PROJECT_ID="qwiklabs-gcp-03-49b42fxxxxxx" # Replace with your GCP Project ID
```

#### 2. Building the file content

```bash
VAR_CONTENT=$(cat << EOF
variable "region" {
  default = "$REGION"
}

variable "zone" {
  default = "$ZONE"
}

variable "project_id" {
  default = "$PROJECT_ID"
}
EOF
)
```

#### 3. This code automatically writes the content into all three variables.tf locations

```bash
echo "$VAR_CONTENT" > variables.tf
echo "$VAR_CONTENT" > modules/instances/variables.tf
echo "$VAR_CONTENT" > modules/storage/variables.tf
```

#### 4. Add the Terraform block and the Google Provider to the main.tf file. Verify the zone argument is added along with the project and region arguments in the Google Provider block.

```bash
cat > main.tf << EOF
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
  zone    = var.zone
}
EOF
```

#### 5.

```bash
terraform init

# verify using

tree .
```

(If the tree utility isn't installed, you can quickly install it with `sudo apt-get install tree -y` or use `find . -maxdepth 3` instead).

Initialize Terraform:

```bash
terraform init
```

---

## Task 2 — Import Existing Infrastructure

Import two existing Compute Engine instances into the instances module and Terraform state.

#### 2a. First, add the module reference into the main.tf file then re-initialize Terraform.

SOLUTION: Add the module reference to main.tf

Open main.tf and add a module block referencing your instances module:

```bash
cat >> main.tf << 'EOF'

module "instances" {
  source     = "./modules/instances"
  region     = var.region
  zone       = var.zone
  project_id = var.project_id
}
EOF
```

Note: >> (double arrow) appends to the file instead of overwriting it — important here since main.tf already has your provider block.

```bash
terraform init
```

##### 2b. Next, write the resource configurations in the `instances.tf` file to match the pre-existing instances.

##### 2bi. Name your instances `tf-instance-1` and `tf-instance-2`.

##### 2bii. For the purposes of this lab, the resource configuration should be as minimal as possible. To accomplish this, you will only need to include the following additional arguments in your configuration: `machine_type`, `boot_disk`, `network_interface`, `metadata_startup_script`, and `allow_stopping_for_update`. For the last two arguments, use the following configuration as this will ensure you won't need to recreate it:

```bash
metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
allow_stopping_for_update = true
```

SOLUTION: Write `instances.tf` inside the module

This defines two `google_compute_instance` resources matching the real VMs, kept minimal as instructed:

```bash
cat > modules/instances/instances.tf << 'EOF'
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "e2-micro"
  zone         = var.zone

  boot_disk {
    initialize_params {
      image = "debian-11-bullseye-v20260714"
    }
  }

  network_interface {
    network = "default"
    access_config {}
  }

  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT

  allow_stopping_for_update = true
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "e2-micro"
  zone         = var.zone

  boot_disk {
    initialize_params {
      image = "debian-11-bullseye-v20260714"
    }
  }

  network_interface {
    network = "default"
    access_config {}
  }

  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT

  allow_stopping_for_update = true
}
EOF

```

IMPORTANT: replace `machine_type` and image values with what you actually found in Step 1 for each instance — they may differ between the two VMs. Also include `access_config {}` inside `network_interface` only if the real instance has an external IP (check in the console/describe output) — most lab default VMs do.

##### 2c. Once you have written the resource configurations within the module, use the terraform import command to import them into your instances module.

SOLUTION: Terraform doesn't know these VMs exist yet — import links your resource blocks to the real infrastructure without recreating them. Run this from your root directory

```bash
terraform import module.instances.google_compute_instance.tf-instance-1 tf-instance-1

terraform import module.instances.google_compute_instance.tf-instance-2 tf-instance-2
```

The syntax is:

```bash
terraform import <module_path>.<resource_type>.<resource_name> <instance_name_in_gcp>
```

Resources imported:

- tf-instance-1
- tf-instance-2

#### 3. Apply your changes. Note that since you did not fill out all of the arguments in the entire configuration, the apply will update the instances in-place. This is fine for lab purposes, but in a production environment, you should make sure to fill out all of the arguments correctly before importing.

```bash
terraform plan

terraform apply
```

Quick sanity check after applying

```bash
terraform state list
```

---

## Task 3 — Configure Remote Backend

### 1. Create a Cloud Storage bucket resource inside the storage module. For the bucket name, use Bucket Name. For the rest of the arguments, you can simply use:

```bash
location = "US"
force_destroy = true
uniform_bucket_level_access = true
```

Note: You can optionally add output values inside of the `outputs.tf` file.

SOLUTION:

##### STEP 1: Check your lab instructions panel for the exact name it wants; if it just says "use a unique name," `var.project_id` is the safe default.

```bash
cat > modules/storage/storage.tf << 'EOF'
resource "google_storage_bucket" "storage-bucket" {
  name                          = "tf-bucket-022682"
  location                      = "US"
  force_destroy                 = true
  uniform_bucket_level_access   = true
}
EOF
```

NOTE: Replace `var.project_id` with the literal string (bucket name string) in quotes instead.

##### Step 2 (optional): Add an output

```bash
cat > modules/storage/outputs.tf << 'EOF'
output "bucket_name" {
  value = google_storage_bucket.storage-bucket.name
}
EOF
```

### 2. Add the module reference to the main.tf file. Initialize the module and apply the changes to create the bucket using Terraform.

SOLUTION: Append a module block for storage, same pattern as you did for instances:

```bash
cat >> main.tf << 'EOF'

module "storage" {
  source     = "./modules/storage"
  region     = var.region
  zone       = var.zone
  project_id = var.project_id
}
EOF
```

Initialize and apply to create the bucket

```bash
terraform init
terraform apply
```

Verify

```bash
gsutil ls
```

### 3. Configure this storage bucket as the remote backend inside the main.tf file. Be sure to use the prefix terraform/state so it can be graded successfully.

SOLUTION

This is the key part — you're telling Terraform to store its state file in this bucket instead of locally. Add a backend "gcs" block inside the existing terraform {} block in main.tf (not a new separate block).
First check your current `main.tf`:

`cat main.tf`

Then edit the terraform {} block to look like this:

```bash
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }

  backend "gcs" {
    bucket = "tf-bucket-022682"
    prefix = "terraform/state"
  }
}
```

IMPORTANT: The bucket value here must be a literal string — Terraform backend configuration can't reference variables (var.project_id won't work here), because backends are configured before Terraform even loads your variables. So type out your actual bucket name as plain text.

### 4. If you've written the configuration correctly, upon init, Terraform will ask whether you want to copy the existing state data to the new backend. Type yes at the prompt.

SOLUTION

```bash
terraform init
```

Quick sanity check

```bash
gsutil ls gs://YOUR-BUCKET-NAME/terraform/state/
```

## You should see a default.tfstate object sitting under that prefix — confirming the remote backend is active and populated.

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

| Name      | CIDR          |
| --------- | ------------- |
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
