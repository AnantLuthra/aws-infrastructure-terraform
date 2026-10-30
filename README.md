# Automating AWS Infrastructure with Terraform

This project automates AWS infrastructure provisioning with Terraform and
deploys an Nginx Docker container on an Amazon Linux EC2 instance.

## What This Project Creates

The project creates and configures:

- A custom VPC
- A subnet in the configured availability zone
- An Internet Gateway
- The VPC's default route table, configured with a route to the Internet
- A default security group with SSH and application access rules
- An EC2 key pair
- An Amazon Linux EC2 instance with a public IP address
- Docker installed and started through EC2 user data
- An Nginx Docker container exposed on port `8080`

## Project Structure

```text
.
├── main.tf
├── providers.tf
├── variables.tf
├── output.tf
├── terraform.tfvars
└── modules
    ├── subnet
    │   ├── main.tf
    │   ├── variables.tf
    │   └── output.tf
    └── webserver
        ├── main.tf
        ├── variables.tf
        ├── output.tf
        └── entry-script.sh
```

## Terraform Modules

### Subnet module

The subnet module creates:

- A subnet
- An Internet Gateway
- A default route table route to the Internet

### Webserver module

The webserver module creates:

- A security group
- An EC2 key pair
- An Amazon Linux EC2 instance
- Docker and Nginx using EC2 user data

## Network Access

The security group allows:

- SSH on port `22` from the IP configured in `my_ip_address`
- Nginx traffic on port `8080`
- All outbound traffic

## Prerequisites

Install and configure:

- [Terraform](https://developer.hashicorp.com/terraform/downloads)
- AWS CLI credentials
- An AWS account with permissions to create the required resources
- An SSH key pair

Configure AWS credentials with:

```bash
aws configure
```

## Configuration

The repository ignores local Terraform variable files and key files. Before
running Terraform, create a root-level `terraform.tfvars` file with values
appropriate for your environment:

```hcl
vpc_cidr_block      = "10.0.0.0/16"
subnet_cidr_block   = "10.0.10.0/24"
avail_zone          = "ap-south-1b"
env_prefix          = "dev"
my_ip_address       = "YOUR_PUBLIC_IP/32"
instance_type       = "t3.micro"
public_key_location = "/path/to/id_ed25519.pub"
image_name          = "al2023-ami-2023.*-kernel-6.18-*"
```

Create or use the referenced SSH public and private key files. For example:

```text
~/.ssh/id_ed25519.pub
~/.ssh/id_ed25519
```

Set `public_key_location` to the location of the public key. The private key
is used locally when connecting to the EC2 instance and is not read by
Terraform.

Use your public IP address for `my_ip_address` so SSH access is restricted to
your network. Do not commit `terraform.tfvars`, private keys, or Terraform
state files.

## Deploy

Initialize Terraform:

```bash
terraform init
```

Review the changes:

```bash
terraform plan
```

Create the infrastructure:

```bash
terraform apply
```

Terraform outputs the public IP address of the EC2 instance.

## Connect to the EC2 Instance

```bash
ssh -i /path/to/private-key ec2-user@<INSTANCE_PUBLIC_IP>
```

After connecting, then verify
Docker:

```bash
docker ps
```

Open the Nginx server at:

```text
http://<INSTANCE_PUBLIC_IP>:8080
```

## Docker Bootstrap

The webserver module uses `modules/webserver/entry-script.sh` as EC2 user data.
The script:

1. Updates Amazon Linux packages
2. Installs Docker
3. Starts Docker
4. Adds `ec2-user` to the Docker group
5. Runs an Nginx container mapped to port `8080`

The Docker group change applies to new login sessions. Reconnect to the
instance before running Docker commands without `sudo`.

## Destroy the Infrastructure

Remove all resources created by Terraform:

```bash
terraform destroy
```

## Security Notes

- Restrict `my_ip_address` to your own public IP.
- Do not allow SSH from `0.0.0.0/0`.
- Do not commit private keys, `terraform.tfvars`, or Terraform state files.
- Port `8080` is currently open to the entire Internet.
- Review the security group rules before using this project in production.

## Notes

- Changing the EC2 user-data script replaces the instance because
  `user_data_replace_on_change` is enabled.
- The `private_key_location` variable is declared but is not currently used by
  the Terraform resources.
