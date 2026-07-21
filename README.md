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
<img width="975" height="587" alt="image" src="https://github.com/user-attachments/assets/28a4e54f-c7c0-4e82-a333-a062b363cc26" />

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
<img width="975" height="378" alt="image" src="https://github.com/user-attachments/assets/2c263159-b8d4-413d-b97b-7d2e56a80c75" />

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
<img width="975" height="372" alt="image" src="https://github.com/user-attachments/assets/2e54cc8c-cdf8-4d84-8b06-961eeb3a58fd" />


## Task 4 — Modify Infrastructure

Update existing VM instances:

Navigate to the instances module and modify the `tf-instance-1` resource to use an `e2-standard-2` machine type.

Modify the tf-instance-2 resource to use an e2-standard-2 machine type.

Add a third instance resource and name it Instance Name. For this third resource, use an e2-standard-2 machine type. Make sure to change the machine type to e2-standard-2 to all the three instances.

Initialize Terraform and apply your changes.

Note: Optionally, you can add output values from these resources in the outputs.tf file within the module.

SOLUTION

Step 1: Check your lab instructions for the third instance's name
Step 2: View your current instances.tf

`cat -n modules/instances/instances.tf`

Step 3: Update the machine type on tf-instance-1 and tf-instance-2

`sed -i 's/machine_type = "e2-medium"/machine_type = "e2-standard-2"/g' modules/instances/instances.tf`

Verify the change:

`grep "machine_type" modules/instances/instances.tf`

Step 4: Add the third instance resource

```bash
cat >> modules/instances/instances.tf << 'EOF'

resource "google_compute_instance" "tf-instance-951871" {
  name         = "tf-instance-951871"
  machine_type = "e2-standard-2"
  zone         = var.zone

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
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

Step 5 (optional): Add outputs

```bash
cat > modules/instances/outputs.tf << 'EOF'
output "instance_1_name" {
  value = google_compute_instance.tf-instance-1.name
}

output "instance_2_name" {
  value = google_compute_instance.tf-instance-2.name
}

output "instance_3_name" {
  value = google_compute_instance.tf-instance-951871.name
}
EOF
```

Step 6: Initialize and apply

```bash
terraform init
terraform plan
terraform apply
```

Quick sanity check afterward

```bash
gcloud compute instances list --format="table(name,machineType.basename(),zone.basename())"
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

1. In the Terraform Registry, browse to the Network Module.

2. Add this module to your main.tf file. Use the following configurations:

- Use version 10.0.0 (different versions might cause compatibility errors).
- Name the VPC VPC Name, and use a global routing mode.
- Specify 2 subnets in the region, and name them subnet-01 and subnet-02. For the subnets arguments, you just need the Name, IP, and Region.
- Use the IP 10.10.10.0/24 for subnet-01, and 10.10.20.0/24 for subnet-02.
- You do not need any secondary ranges or routes associated with this VPC, so you can omit them from the configuration.

SOLUTION

```bash
cat >> main.tf << 'EOF'

module "vpc_network" {
  source  = "terraform-google-modules/network/google"
  version = "10.0.0"

  project_id   = var.project_id
  network_name = "tf-vpc-860487"
  routing_mode = "GLOBAL"

  subnets = [
    {
      subnet_name   = "subnet-01"
      subnet_ip     = "10.10.10.0/24"
      subnet_region = var.region
    },
    {
      subnet_name   = "subnet-02"
      subnet_ip     = "10.10.20.0/24"
      subnet_region = var.region
    }
  ]
}
EOF
```

NOTE: Replace "VPC_NAME" with the exact name your lab expects.

The terraform-google-modules/network/google module expects subnets as a list of objects, each needing at minimum subnet_name, subnet_ip, and subnet_region — which matches exactly what the task says you need (Name, IP, Region), so no extra arguments like secondary ranges are required.

3. Once you've written the module configuration, initialize Terraform and run an apply to create the networks.

`terraform init`

Explanation after terraform init

- Pull from Terraform Registry (not just a local ./modules/... path)

```bash
terraform plan
terraform apply
```

Verify:

```bash
gcloud compute networks subnets list --filter="region:$(gcloud config get-value compute/region)"
```

4. Next, navigate to the instances.tf file and update the configuration resources to connect tf-instance-1 to subnet-01 and tf-instance-2 to subnet-02.

SOLUTION

```bash
cat -n modules/instances/instances.tf
```

For `tf-instance-1`, change its `network_interface` block to:

```bash
network_interface {
    network    = "tf-vpc-860487"
    subnetwork = "subnet-01"
    access_config {}
  }
```

For `tf-instance-2`, change its `network_interface` block to:

```bash
network_interface {
    network    = "tf-vpc-860487"
    subnetwork = "subnet-02"
    access_config {}
  }
```

Note: Within the instance configuration, you will need to update the network argument to VPC Name, and then add the subnetwork argument with the correct subnet for each instance.

```bash
terraform plan

terraform apply
```

Important thing to check here: changing the network/subnetwork of an existing instance's network_interface is one of those changes that typically forces recreation (destroy + create) rather than an in-place update — unlike the machine type change earlier.

Quick sanity check afterward

```bash
gcloud compute instances describe tf-instance-1 --zone=$(gcloud config get-value compute/zone) --format="value(networkInterfaces[0].subnetwork)"
gcloud compute instances describe tf-instance-2 --zone=$(gcloud config get-value compute/zone) --format="value(networkInterfaces[0].subnetwork)"
```

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
<img width="975" height="670" alt="image" src="https://github.com/user-attachments/assets/3ae82bee-da46-488c-9b4f-0b5d44000c23" />

<img width="975" height="490" alt="image" src="https://github.com/user-attachments/assets/88e02c4b-a1c9-4633-ac8a-47f08fea478c" />


## Task 7 — Configure Firewall

Create a firewall rule allowing HTTP traffic.

Configuration:

- TCP Port 80
- Source Range

```
0.0.0.0/0
```

SOLUTION

Step 1: Find the network's self_link/ID from Terraform state

```bash
terraform state show module.vpc_network.google_compute_network.network
```

Look for the self_link or id field in the output — it'll look like:

```bash
projects/YOUR_PROJECT_ID/global/networks/VPC_NAME
```

You can also list all outputs the module provides:

`terraform state list | grep vpc_network`

Step 2: Create a firewall rule resource in the main.tf file, and name it tf-firewall.
This firewall rule should permit the VPC Name network to allow ingress connections on all IP ranges (0.0.0.0/0) on TCP port 80.
Make sure you add the source_ranges argument with the correct IP range (0.0.0.0/0).

```bash
cat >> main.tf << 'EOF'

resource "google_compute_firewall" "tf-firewall" {
  name    = "tf-firewall"
  network = module.vpc_network.network_self_link

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  source_ranges = ["0.0.0.0/0"]
}
EOF
```

`----network = module.vpc_network.network_self_link` — this is the cleanest way to reference the network, since it pulls the value directly from the module's output rather than hardcoding a string. The `terraform-google-modules/network/google` module exposes `network_self_link` as a standard output, so this should resolve correctly without you needing to hand-type the project ID/VPC name.
-- If for some reason that output name doesn't exist in your module version, check available outputs with:

```bash
terraform state show module.vpc_network.google_compute_network.network
```

```bash
terraform state list
```

Initialize Terraform and apply your changes.

```bash
terraform init
terraform plan
```

Note: To retrieve the required network argument, you can inspect the state and find the ID or self_link of the google_compute_network resource you created. It will be in the form projects/PROJECT_ID/global/networks/VPC Name.

Step 4: Verify

```bash
gcloud compute firewall-rules describe tf-firewall --format="yaml(name,network,sourceRanges,allowed)"

terraform state list
```
<img width="975" height="472" alt="image" src="https://github.com/user-attachments/assets/89e9c962-1e9c-48e3-b923-1c6ba502946d" />

<img width="975" height="370" alt="image" src="https://github.com/user-attachments/assets/7b0fbeb7-3041-4236-aca7-2b27282a0ba8" />




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

## Compiled By

**Segun Ofe**

Cloud Architect | DevOps Engineer | Terraform Associate | Google Cloud Professional Cloud Architect | AWS Solutions Architect Associate

GitHub: https://github.com/segunofe

LinkedIn: https://linkedin.com/in/segunofe
